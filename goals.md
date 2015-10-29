© (C) Copyright 2015 Travis Rigg
© Copyright phpBB Limited 2003-2014
This project is distributed under the terms of the GNU General Public License v2

This file is part of SFIM.

    SFIM is free documentation: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 2 of the License, or
    (at your option) any later version.

    SFIM is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with SFIM.  If not, see <http://www.gnu.org/licenses/>.

# SFIM
SFIM stands for Standard Forum Interaction Modeling. The purose of the SFIM
project is to create an ecosystem in which forums have a high level of
interoperability. The idea is that any forum created using SFIM will have a well
documented set of interactions that will be consistent. This way a forum
administrator can port their interface manager from one forum manager software
to another without having to also move data from one database to another. This
way an android app developer can create a generic app that can be used to log
into any SFIM forum, and a user can select any app targeting SFIM forums to
access one or more of their favorite forums.

## Entity-Relationship Model
The very most important component of SFIM will be the Entity-Relationship
Model, or ER Model, describing how databases using SFIM should be
structured. This will incude as much description as possible of what the ER
Model is trying to convey. The reason for this is that any implementation of
SFIM will be based upon this ER Model, and can be thought of as an example of
SFIM. Any implentations of SFIM that are part of the SFIM project are meant to
be reference implemtations. These are not meant to be the official
implementation, best implementation, or most popular implementation. They are
meant to be examples of how things are done that are also usable directly.
Potential users and developers are encouraged to reimplement any aspect of the
referenece implementations as they see fit. However, they must keep in mind that
they are advised to follow the ER Model as closely as possible so that it will
be easier for other developers to make their components work together.

The ER Model will be available to anyone under the terms of the GPL v2.0

### Compatability With Others
It is important that forum administrators choosing to switch from one forum
management system to a SFIM management system be able to port their forum
easily. For this reason the ER Model will be similar to other popular forum
management systems.

## Database Management System
The reference implementation Database Management System, or DBMS, will be
written in SQL. The reason for this is that SQL is an extremely widespread
language, known by many developers, designed to be as human readable as
possible. However, certain web application programming frameworks have methods
for creating and interacting with databases without ever writing a single line
of SQL. This is part of the advantage of SFIM. A programmer is free to use any
implementation of any component that they feel is best for their needs,
including creating a new implementation of the entire stack.

The implementation of the DBMS will be available under an MIT license

## Web Interface
The reference implementation of a web interface using SFIM will be written in
Python using Pyramid.

The implementation of the web interface will be available under an MIT license

### Why Python?
Python is an extremely readable language that is known by a great many
programmers. For many programmers, it was their first language. This means that
it is known by an even larger number of programmers than perhaps just the
programmers who use it.

#### Why Pyramid?
Pyramid was designed to allow the programmer to choose how they want to store
their database. This fits in extremely well with the goals of SFIM, especially
for a reference implementation.

#### Why Not Django?
Django was second choice after Pyramid. The reason for this is that Django is an
extremely well known framework, used by many developers. However, it does not
allow for the same level of flexability that Pyramid does.

### Why not Ruby, PHP, Perl, or one of the other popular web languages?
Many of these languages provide the programmer with the ability to do things
with the minimum amount of effort coding. However, this can sometimes leave
things confusing for other programmers, and can be especially difficult for
new programmers. The reference implementations of SFIM are meant to be as clear
as possible so that it is absolutely clear and explicit at all times what tasks
the computer is performing.

Anyone who wants to create an implementation of the SFIM web interface in
another framework or language than the reference implementation is encouraged to
do so. If you think you can build a better implementation, you probably can.

## Android App
There are plans to build a reference android app. The capabilites of this app
would be to log into and interact with forums, and to allow multiple accounts on
multiple sites to be logged in. Further details about the exact specifications
of this android app will be given as the project grows and as the lead developer
learns how to create apps for android.

The implementation of an Android App will be available under an MIT license.