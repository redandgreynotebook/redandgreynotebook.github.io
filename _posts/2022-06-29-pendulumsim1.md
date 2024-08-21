---
title: pendulum simulator
tags: light project pendulumsim
math: true
---

so one of my youtube rabbit holes led me to this: 

<div>{%- include embed/youtube.html id='d0Z8wLLPNE0' -%}</div>

so i thought i'd try and make a pendulum physics simulator of my own. in order to do that, i needed two things:
<!--more-->
* a diffeq solver to solve motion equations, and 
* a graphics library to draw my pendulums in motion.

fortunately, c++ has both of these: the library [`odeint`](https://www.boost.org/doc/libs/1_79_0/libs/numeric/odeint/doc/html/index.html) for diffeq solving, and openGL for graphics. 

### drawing a pendulum

is pretty simple: in its most simplest form, a pendulum's just a thin rod, which i can represent with a rectangle. to make it look better, i *could* draw a cicle at the the end of the pendulum, but that's just for visual purposes and doesn't take anything away from the actual simulation. after setting up openGL's vertex array object, drawing a rectangle is pretty easy: 

```c++
// coordinates for the rectangle
float vertices[] = {
	 0.002f,  0.5f, 0.0f,  // top right
	 0.002f, -0.5f, 0.0f,  // bottom right
	-0.002f, -0.5f, 0.0f,  // bottom left
	-0.002f,  0.5f, 0.0f   // top left 
};
unsigned int indices[] = {  // note that we start from 0!
	0, 1, 3,  // first Triangle
	1, 2, 3   // second Triangle
};

// all the necessary openGL setup, like shader compliation and buffer setup, goes here

// ourShader's, well, our shader object, both vertex and fragment shader in one
ourShader->use();
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

now for making it swing. in order to do that, we'll need the pendulum's angle (from the vertical) $$\theta$$. we need to be able to rotate our rod. let's use a matrix to represent the transformations we want to do on the rod, since openGL's `glm` library has matrix transformation functions. what we want to do is:

* translate our rod so that the top is at the origin, then
* rotate our rod by $$\theta$$, and then
* translate it back to our original position. 

```c++
// trans is our 4x4 transformation matrix
ourShader->use();
trans = glm::mat4(1.0f);

// transformation order matters
// theta is our angle
trans = glm::translate(trans, glm::vec3(0.0f, 0.5f, 0.0f));
trans = glm::rotate(trans, theta, glm::vec3(0.0f, 0.0f, 1.0f));
trans = glm::translate(trans, glm::vec3(0.0f, -0.5f, 0.0f));

// load our matrix into our shader
unsigned int transformLoc = glGetUniformLocation(ourShader->ID, "transform");
glUniformMatrix4fv(transformLoc, 1, GL_FALSE, glm::value_ptr(trans));

// and now draw
glBindVertexArray(VAO);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

transformation order matters, and here's why: matrix multiplication does *not* commute. if we read our code line-by-line, and let $$A$$ be the matrix of the first transformation we perform (translating up by 0.5) and $$B$$ and $$C$$ be the second and third respectively, our final transformation will evaluate to $$ABC\vec v,$$ where $$\vec v$$ is the vector we transform by. since matrix multiplication is associative, we can think of it as $$A\left(B\left(C\vec v\right)\right)$$:

* first, multiplication by $$C$$ translates it down 0.5 (to the origin), then
* multiplication by $$B$$ rotates it, and then
* multiplication by $$A$$ translates it back to its original position.

in openGL, you code your transformations you want to perform in the *opposite* order in which you thought of them; for example, we wanted to translate it down first, so we write that line last. 

### solving motion equations 

before we tackle fancy stuff, we demonstrate with a simple pendulum, with the motion equation 

$$\theta ''(t) = -\frac{g \sin\left(\theta(t)\right)}{L}$$

where $$g$$ is the force of gravity and $$L$$ is the pendulum's length. for sake of simplicity we let $$L = 1$$. this is a second-order (there are two ' on the left hand side) and `odeint` only handles first-order (so only first derivatives), so we let $$u = \theta '(t)$$ and create a system that `odeint` can understand: 

$$\begin{align*}
\theta'(t) &= u(t) \\
u'(t) &= -g \sin\left(\theta(t)\right)
\end{align*}$$

```c++
typedef std::vector< float > state_type;

// dxdt[0] is theta'(t), dxdt[1] is u'(t), which is theta''(t)
// x is the array that represents our state, so x[0] is theta(t), while x[1] is u(t)
void real_pendulum(const state_type& x, state_type& dxdt, const double /* t */) {
	dxdt[0] = x[1];
	dxdt[1] = -gravity * sin(x[0]);
}
```

in order to simulate time, we need to keep track of time in our openGL render loop. we do this by storing the time of our previous frame (the last time our loop executed) and the time of our current frame; this way we can find the change in time between them, which is important when we're given a diffeq with initial conditions at time 0. then, we can use `odeint::integrate()` function to solve it, given our initial value:

```c++
// this is at the start of int main()
float lastFrame = 0.0f;
float deltaTime = 0.0f;

// from here on down we're in our render loop

// get changes in time
float time = (float) time_constant * glfwGetTime();
deltaTime = time - lastFrame; 
lastFrame = time;

// so, the arguments of this function, in order: 
// 1) real_pendulum is a function pointer to our function defined previously
// 2) theta is a vector of 2 floats that keeps track of our current state
// x[0] is theta(t), x[1] is u(t)
// 3) the initial time we start at
// 4) the final time
// 5) our step size, more steps corresponds to more accurate results
boost::numeric::odeint::integrate(real_pendulum, theta, time - deltaTime, time, deltaTime / NUM_STEPS);
```

put this together, and you get a nice, simple pendulum animation. 

### now with some class

but what if you want multiple pendulums? creating multiple pendulums this way would be a pain. this is where we use classes. between different types of pendulums -- simple, double, [elastic](https://en.wikipedia.org/wiki/Elastic_pendulum), single, [coupled](https://www.maths.surrey.ac.uk/explore/michaelspages/documentation/Coupled), engaged, married, it's complicated -- they all share a lot in common:

* a system of diffeqs as their motion equation
* a particle trail (we'll get to this later)
* shaders to actually draw pendulums
* vertex data to draw the rod
* a transformation matrix to animate the rod

so, given all the different types of pendulums we might make, it's best to create an abstract `Pendulum` class and let subclasses represent different types of pendulums. since each pendulum rod is the same, we can make our buffer objects `static` since each instance uses the same base rod (they just transform it differently). 

```c++
typedef std::vector< float > state_type;

class Pendulum {
public:

Pendulum(Shader* o_shader, Shader* p_shader, state_type state, glm::vec3 color);

static void init(); // sets up static variables and buffer objects

virtual void update(float t, float dt) = 0; // solves diffeq and adds particles
virtual void draw() = 0; // actually draws rod and particles
virtual glm::vec2 location() = 0; // gets location of tail of pendulum (particle pos)

protected:

// things for drawing the actual bar -- 
// these are shared resources since matrix transformations make up the actual changes

static unsigned int VAO, VBO, EBO;
static float vertices[12];
static unsigned int indices[6];

// useful constants 

static int n_steps; // number of steps when we solve diffeq
static float gravity;

// stuff for particle trail

static int n_particles;
static float decay;

// at the beginning, freeze time for initial_delay seconds before starting at t = 0

static float initial_delay;

// ParticleGenerator is for particle trail

ParticleGenerator* particles;

glm::mat4 trans; // transformation matrix
glm::vec3 color; // pendulum color
state_type state; // pendulum state, whatever that might mean

// shader objects: the first draws the rod, the second draws particles

Shader* object_shader, * particle_shader;
};
```

now to look at function overrides of the subclass `SpringPendulum`, which is the elastic pendulum we mentioned earlier. we have to solve [this sytem](https://en.wikipedia.org/wiki/Elastic_pendulum#Equations_of_motion) of diffeqs, which we can convert into first-order in similar fashion. at any given time $$t$$, the spring pendulum has length $$1 + x(t)$$ and angle $$\theta(t)$$; calculating `location()` is a matter of right triangle trig. 

```c++
// this is the diffeq we have to solve
// x[0] is x(t)
// x[1] is x'(t)
// x[2] is theta(t)
// x[3] is theta'(t)
// and dxdt[i] is x'[i] if that makes sense
// (so dxdt[0] is x'(t), dxdt[1] is x''(t), and so on)
void SpringPendulumDiffeq::operator() (const state_type& x, state_type& dxdt, const double /* t */) {
	dxdt[0] = x[1];
	dxdt[1] = (1.0f + x[0]) * x[3] * x[3] - spring_constant * x[0] + gravity * cos(x[2]); // mass 1
	dxdt[2] = x[3];
	dxdt[3] = -gravity * sin(x[2]) / (1.0f + x[0]) - 2 * x[1] * x[3] / (1.0f + x[0]);
}

void SpringPendulum::update(float time, float dt) {
	time -= initial_delay;
	if (time >= 0.0f) {
		// solve diffeq and add particles
		boost::numeric::odeint::integrate(*trajectory, state, time - dt, time, dt / n_steps);
		particles->Update(dt, location(), 1);
	}

	// update position
	trans = glm::mat4(1.0f);
	trans = glm::translate(trans, glm::vec3(0.0f, 0.5f, 0.0f));
	trans = glm::rotate(trans, state[2], glm::vec3(0.0f, 0.0f, 1.0f));
	trans = glm::scale(trans, glm::vec3(1.0f, 1.0f + state[0], 1.0f));
	trans = glm::translate(trans, glm::vec3(0.0f, -0.5f, 0.0f));
}

void SpringPendulum::draw() {
	this->object_shader->use();
	this->object_shader->setVector3f("ourColor", color);
	this->object_shader->setMatrix4("transform", trans);
	glBindVertexArray(this->VAO);
	glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
	glBindVertexArray(0);

	particles->Draw();
}

glm::vec2 SpringPendulum::location() {
	return glm::vec2((1.0f + state[0]) * sin(state[2]), 0.5f - (1.0f + state[0]) * cos(state[2]));
}
```

running `SpringPendulum::update()` and `SpringPendulum::draw()` openGL's render loop will give us our spring pendulum simulation. 

### particles

the spring pendulum's trajectory is pretty chaotic; it moves faster or slower based on the length of the pendulum at the time. let's try and create a trail that traces out our pendulum's recent trajectory. so during each frame, let's create a particle (so, just a little dot or something) at the pendulum's current location. this particle would have a certain lifespan, and as time passes the particle slowly ages until it "dies" and is no longer rendered. 

```c++
// Represents a single particle and its state
struct Particle {
    glm::vec2 Position;
    glm::vec4 Color;
    float     Life;

    Particle() : Position(0.0f), Color(1.0f), Life(0.0f) { }
    Particle(glm::vec4 color) {
        Position = glm::vec2(0.0f);
        Life = 0.0f;
        Color = color;
    }
};
```

now let's write a `ParticleGenerator` class that stores, updates, and draws all our particles. before writing methods, let's gather what we want this class to do every render loop first, in order:

* "age" all particles by reducing their `Life` and increasing their transparency
* create a `Particle`, newly born with full `Life` and opacity
* draw all particles, not rendering them if `Life` is not positive

we store a bunch of `Particle` instances in a `vector`. at any given moment, the particles we store occupy a contiguous (maybe wrapping around, so imagine starting near the end of the vector and looping back to 0, 1, 2, 3...) subarray of our vector. so, there exists an index immediately to the right of our contiguous subarray that represents the first free index where we can create a new `Particle`. the first two steps we want can be implemented below. (`firstUnusedParticle()` finds our aforementioned index.)

```c++
void ParticleGenerator::Update(float dt, glm::vec2 position, unsigned int newParticles, glm::vec2 offset)
{
    // add new particles 
    for (unsigned int i = 0; i < newParticles; ++i)
    {
        int unusedParticle = this->firstUnusedParticle();
        this->respawnParticle(this->particles[unusedParticle], position);
    }
    // update all particles
    for (unsigned int i = 0; i < this->amount; ++i)
    {
        Particle& p = this->particles[i];
        p.Life -= decay * dt; // reduce life
        if (p.Life > 0.0f)
        {	// particle is alive, thus update
            p.Color.w -= dt * decay;
        }
    }
}
```

and now drawing them is easy: just set up appropriate vertex data and buffer objects, then use your shader and you're good to go.

---

i'll be back with actual videos of my animation. 