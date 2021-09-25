---
layout: single
title:  "Changing Area and Faraday's Law"
toc: true
categories: 
  - physics
tags: 
  - E&M
header:
  overlay_image: /assets/images/Figure_1.png
  overlay_color: "#333"
  overlay_filter: 0.5
excerpt: An introductory physics problem isn't as straightforward as it seems
---
Consider the classic introductory problem where a bar is moving over a constant magnetic field. What is the generated EMF \\(\varepsilon\\)?
![A Moving Bar](/assets/images/movingbar.png){: .align-center}  

## The Standard Solution
Using Maxwell's equations and Lenz's law, EMF is given as
$$ \varepsilon = - \frac{d\Phi}{dt} $$  
where \\(\Phi=\int B\cdot dA=BA\\). Because \\(B\\) is constant and the area is changing we get that 

$$ \boxed{\varepsilon = -\ell B v }$$  

## Less Obvious Solution
Looking at Maxwell's equations we have  

$$ \nabla \times E= - \frac{\partial B}{\partial t} $$  

However, \\(\frac{\partial B}{\partial t}=0\\), which seems to imply \\(E=0\\) here! To rectify this, we notice that it implies the electric field is a constant not zero, which makes sense given the EMF given above. It's also important to notice that you cannot move the time derivative past the integral sign without the chain rule, as the area part of the integral is time dependent.
