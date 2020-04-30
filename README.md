# Gadgetron Online Course

During June 2020 we will host the “Gadgetron Online Course”. Although initially scheduled to take place in Bordeaux as a Summer School, the COVID-19 situation made that impossible. We are instead hosting it online.
 

This course is aimed at both new and experienced users of Gadgetron, covering basic reconstruction as well as the latest functionalities. The topics covered are intended for researchers in basic science and/or clinical research. 


The course is organised remotely and will be accessible in live via **visio-conference**. In addition of the course and pratical course, participants will have the possibility to propose work on a MRI reconstruction topic during **interactive session** with the speakers. Please, go to this [page](https://github.com/gadgetron/GadgetronOnlineClass/tree/master/Interactive-Sessions) for more information, we encourage you to apply.


The Gadgetron Online Course is made of several modules that constitute together the scientific stack. Gadgetron made use of standard computing languages (C++, Cmake, CUDA, Python, Matlab) that are themselve calling hundreds of libraries/packages/functions. We won't cover everything in this short course, but you should get enough information to decide if your research can benefit from Gadgetron. And I bet it will likely do.


For any questions, feel to contact us on the [forum](https://groups.google.com/forum/#!forum/gadgetron) of Gagdetron or at gadgetron2020 /at/ sciencesconf.org


Organizers: David Hansen, Kristoffer Knudsen, Hui Xue, Oliver Joseph, Vinai Roopchansingh, John Derbyshire, Aurélien Trotier, Stanislas Rapacchi, Maxime Yon, Pierre Bour, Valéry Ozenne



## Agenda

### Day 1 : Gadgetron Introduction

Date  | Time | Link | Topic | Tutor
----- | ---- | ----- | ----- | -----
June 11, 2020 | 14:00-15:00 | [link](Courses/Day1/Lecture1) | [Gadgetron, a high level overview introduction] | David Hansen
June 11, 2020 | 15:00-16:00 | [link](Courses/Day1/Lecture2) | [Gadgetron, a low level overview introduction] | Kristoffer Knudsen
June 11, 2020 | 16:00-17:00 | [link](Courses/Day1/Lecture3) | [Basic reconstruction using Python] | Valery Ozenne

### Day 2 :Ismrmrd, a vendor-free format for MRI reconstruction

Date  | Time | Place | Topic | Tutor
----- | ---- | ----- | ----- | -----
June 18, 2020 | 14:00-15:00 | [link](Courses/Day2/Lecture1) | [Ismrmrd Part 1: Introduction] | Maxime Yon  
June 18, 2020 | 15:00-16:00 | [link](Courses/Day2/Lecture2) | [Ismrmrd Part 2: Xml style sheet : Siemens/GE/Bruker conversion] | Vinai & John
June 18, 2020 | 16:00-17:00 | [link](Courses/Day2/Lecture3) | [Communication process with the Siemens scanner] | Hui Xue , Pierre ? 

### Day 3 : Programming lessons

Date  | Time | Place | Topic | Tutor
----- | ---- | ----- | ----- | -----
June 25, 2019 | 14:00-15:00 | [link](https://link) | [Protyping at the scanner with Matlab ] | Oliver Josephs ?  
June 25, 2019 | 15:00-16:30 | [link](https://link) | [ ] |  
June 25, 2019 | 16:00-17:00 | [link](https://link) | [ ] |  


## Participate to Online Course Agenda (Preparation and Modalities)

> <img src="https://img.shields.io/badge/-_Warning-orange.svg?style=flat-square"/>
> Note that this course is based on the following teaching material: 

To achieve good interaction between lecturers and participants - and especially tutors and participants in the work in progress sessions -  we highly recommand to read the preparation list.

Preparation list for speakers is available [here](Preparation).

Preparation list for participants [here](Preparation). 	


## Website Structure

All information will be written in a README file in each directories. Additionnal contents like data, codes, powerpoint will be uploaded or indicated in the material folder for each lecture 

```bash

├── Courses
│   ├── Day1
│   │   ├── Lecture1
│   │   │   └── README.md
│   │   ├── Lecture2
│   │   │   └── README.md
│   │   └── Lecture3
│   │       └── README.md
│   ├── Day2
│   └── Day3
├── Installation
│   └── README.md
├── Interactive-Sessions
│   └── README.md
├── Preparation
│   └── README.md
└── README.md

```


## Test the Ismrmrd and Gadgetron installation in advance

Since all participants are working at home on their own computer, we asked the participants to test their ISMRMRD and Gadgetron installation in advance. 
Detailed installation instructions has been summarized [here](https://github.com/gadgetron/GadgetronOnlineClass/tree/master/Installation).  

Feel free to contact us and to post any inquiries on the gadegtron [forum](https://groups.google.com/forum/#!forum/gadgetron)

> <img src="https://img.shields.io/badge/-_Warning-orange.svg?style=flat-square"/>
> Note that this course is based on the following teaching material: 
> [link](http://gadgetron.github.io/tutorial/) 
> [link](https://github.com/gadgetron/gadgetron/wiki/Gadgetron-Hello-World)
> Running one of them before is highly recommended

## 1. Introduction courses (day 1 : June 11th 2020)

### 1.1 - Gadgetron, a high level overview introduction(14:00 -> 15:00 CEST)

This [lecture](introduction-part1.md) does not attempt to be comprehensive and cover every single feature, or even every commonly used feature. Instead, it introduces many of Gadgetron's most
noteworthy features, and will give you a good idea of the Gadgetron's capability and
usage.

**See also**:

 * [wiki](https://github.com/gadgetron/gadgetron/wiki/Gadgetron-Gadgets)


### 1.2 - Introduction (15:00 -> 16:00 CEST)

This [gentle introduction to Gadgetron](introduction-part1.md) explains .

**See also**:

 * [Wiki](https://github.com/gadgetron/gadgetron/wiki/Gadgetron-Streaming-Architecture)

### 1.3 - Basic reconstruction using Python  (16:00 -> 17:00 CEST)

The primary goal of this [lecture](https://github.com/gadgetron/GadgetronOnlineClass/tree/master/Material/Day1/Lecture3) introduces the python gadget and the ismsmrd-python-toolboxes
 that contains various toolboxes dedicated to common issues. Its different submodules correspond to different applications, such as fourier transfrom, coil sensitivity map estimation, grappa reconstruction, etc.

**See also**:

  * []()
  * []()
  * []()



## 2. Toward the scanner room  (day 2: June 18th)

### 2.1 - Title  (  )

This [lesson]() introduces ...

**See also**:

  * [ ]( )
  * [ ]( )


### 2.1 - Title  (  )

This [lesson]() introduces ...

**See also**:

  * [ ]( )
  * [ ]( )


### 2.1 - Title  (  )

This [lesson]() introduces ...

**See also**:

  * [ ]( )
  * [ ]( )











<!----------------------------- External links ------------------------------->
[Git]:        https://git-scm.com
[Docker]:     https://docs.docker.com/get-docker/
[Python]:     http://www.python.org
[Numpy]:      http://www.numpy.org
[Scipy]:      http://www.scipy.org
[Matplotlib]: http://matplotlib.org



<!---------------------------------------------------------------------------->
