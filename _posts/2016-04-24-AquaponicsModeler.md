---
layout: post
title: "Aquaponics Modeler: a modeling application for Aquaponics systems."
category: project
date: "2016-04-24"
tags: 
    - python
    - qt
    - desktop-application
    - aquaponics
    - modeling
    - documentation
    - sphinx
---
Little by little I am working at my current job on an [Aquaponics system](http://www.theaquaponicsource.com/what-is-aquaponics/) together with the help of our volunteers.
As a final step to the first implementation I need to regulate the flow of water from our fish tank to the grow bed. I decided to do this with a timer that switches the pump on and off. In order to create the electronics for this timer, I need to know accurately how much water is flowing from one compartment into the next.
Of course I could just have done the maths for this on the back of a napkin, but I though writing an application that does the simulation would be much more fun.

* ![Aquaponics Modeler Screenshot](/images/aquaponics_modeler_2.png)
* ![Aquaponics Modeler Screenshot](/images/aquaponics_modeler_1.png)

And as it turns out, one thing leads to another, and before you know it you have a complete desktop application with [proper documentation](http://allican.be/AquaponicsModeler/) that can be used by anyone. It was fun to do and is helping me create our own aquaponics setup. I don't know if it is of any use to anyone else, but for anyone interested, the documentation can be found [on my website](http://allican.be/AquaponicsModeler/) and the code on [Github](http://www.github.com/dolfandringa/AquaponicsModeler/).

The application is written using the [Qt application framework](http://doc.qt.io/qt-5/), making use of [PyQt5](https://www.riverbankcomputing.com/software/pyqt/intro). The plots are being generated my the [matplotlib.pyplot](http://matplotlib.org/users/pyplot_tutorial.html) library. The documentation has been created using [Sphinx](http://www.sphinx-doc.org/) with the autodoc, apidoc and [napoleon](http://sphinxcontrib-napoleon.readthedocs.org/en/latest/) extensions. This allows me to write the documentation in docstrings using the Google syntax and generate the corresponding html files automatically. Codeanchaos wrote [a great howto on sphinx, apidoc and autodoc](https://codeandchaos.wordpress.com/2012/07/30/sphinx-autodoc-tutorial-for-dummies/).
