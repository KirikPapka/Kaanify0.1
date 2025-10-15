# Kaanify0.1


# Mathematical Principles Behind `Kaanify`

This simulation merges **computational geometry**, **fluid dynamics**, and **optimization** into a dynamic morphing system that reconstructs a portrait from a blank canvas.

## 1. Overview

The algorithm evolves a field of *seeds* and a *fluid velocity field* to minimize the image energy:

$$
E = \int_\Omega \big(I(x,y) - T(x,y)\big)^2 \, dx\,dy
$$

where \( I(x,y) \) is the current generated image and \( T(x,y) \) is the target portrait.

---

## 2. Jump Flood Algorithm (JFA)

JFA approximates a **Voronoi diagram**: each pixel \( p \) takes the nearest seed \( s_i \) by Euclidean distance

$$
s^*(p) = \arg\min_{s_i} \,\|\,p - \mathbf{x}_{s_i}\,\|^2.
$$

The pixel’s gray equals the owning seed’s gray. This is a fast multi-pass approximation of the Euclidean distance transform.

---

## 3. Morphing (Color Relaxation)

Each seed \( i \) has current gray \( g_i(t) \), a target gray \( g_i^* \), and activation \( a_i \in [0,1] \).  
We update by explicit Euler (relaxation):

$$
g_i(t+\Delta t) = g_i(t) + a_i\,\lambda\,\big(g_i^* - g_i(t)\big),
$$

which discretizes the ODE

$$
\frac{dg_i}{dt} = a_i \lambda \big(g_i^* - g_i\big).
$$

Active seeds exponentially approach their targets.

---

## 4. Activation Diffusion (Optional)

If activation spreads, it follows the **heat equation** in discrete form:

$$
a'(x,y) = a(x,y) + D\,\nabla^2 a(x,y),
\qquad
\frac{\partial a}{\partial t} = D\,\nabla^2 a.
$$

---

## 5. Fluid Dynamics (Navier–Stokes Approximation)

We simulate an incompressible 2D velocity field \( \mathbf{v}(x,y,t) \) with viscosity \( \nu \) and pressure \( p \):

$$
\begin{cases}
\dfrac{\partial \mathbf{v}}{\partial t} + (\mathbf{v}\cdot\nabla)\mathbf{v}
= -\nabla p + \nu \nabla^2 \mathbf{v} + \mathbf{f}, \\[6pt]
\nabla \cdot \mathbf{v} = 0.
\end{cases}
$$

### Error-Driven Forcing

The external force reduces image error by following the gradient of the difference:

$$
\mathbf{f}(x,y) = -\,k\,A(x,y)\,\nabla\!\big(I(x,y) - T(x,y)\big),
$$

where \( A(x,y) \) is an activation mask (seeds you’ve “painted”).

---

## 6. Pressure Projection (Poisson Solve)

To enforce incompressibility, we solve the **Poisson equation** for pressure and project:

$$
\nabla^2 p = \nabla\cdot\mathbf{v},
\qquad
\mathbf{v} \leftarrow \mathbf{v} - \nabla p,
$$

implemented via Jacobi iterations in the code.

---

## 7. Vorticity Confinement

To restore swirl lost to numerical diffusion, we add:

$$
\mathbf{v} \leftarrow \mathbf{v} + \epsilon \,\big(\nabla |\omega| \times \hat{z}\big)\,\omega,
\qquad
\omega = \nabla\times\mathbf{v}.
$$

---

## 8. Seed Advection + Spring Constraint

Seeds move with the flow and are softly pulled toward a rest grid (to avoid holes):

$$
\mathbf{x}_i \leftarrow (1-w)\Big(\mathbf{x}_i + \mathbf{v}(\mathbf{x}_i)\,\Delta t\Big) + w\,\mathbf{x}_i^{\text{rest}}.
$$

---

## 9. Numerical Building Blocks

- **JFA / Voronoi ownership:** multi-pass nearest-seed propagation  
- **Advection:** semi-Lagrangian backtrace sampling  
- **Diffusion:** discrete Laplacian smoothing  
- **Pressure:** Jacobi iterations for Poisson equation  
- **Morphing:** explicit Euler relaxation
