---
published: false
layout: post
title: "Engineering Teams"
date: 2014-12-18 21:01:50 -0800
comments: true
categories:
---
Househappy has five small engineering teams delineated by expertise: Backend,
Web Front End, iOS, DevOps and QA.

The Backend team focuses on the server-side systems, including the API and the
data import system.

The Web Front End team designs, develops and maintains the Spine JavaScript
application and the automated emails. The Spine application is deployed
directly to a CDN, and it has no direct dependency on the Rails code that the
API is written in.

The iOS team develops and maintains the Househappy iOS app. The iOS app is
written in Objective C.

In addition to the application development teams, Househappy has a DevOps team
that manages its server infrastructure. Part of the skill set of DevOps is
writing Chef recipes in Ruby.

Each software team has their own code repository or repositories. Each
repository is deployed independently from one another. This is different from
some other organizations, where there might be a single single code base shared
by the back end and front end teams.

It is helpful to have independence between the teams' code bases. Different
teams can work more independently from one another. Deployments can be isolated
to only one team's work, which reduces the complexity of testing and QA.

Even though code repositories are separate, members from a team can submit
changes to the repository of another team. Code reviews ensure than the
submitted code adheres to standards. This also gives engineers experience
working with different languages and technologies. These skills can become
leverage for switching to another team, or it can be just something valuable to
put on their résumé. In the short term, it's a way to get their task done
without having to wait for another team to implement a portion of it.

The QA team manages testing of all the teams' work.

The Backend and Web Front End teams both have daily standing meetings, called
"standups". These short meetings take place at the beginning of the day.
Each team member reports on what they worked on the previous day, what they
are working on today, and what obstacles they are encountering, if any. This
gives everybody a change to keep aware of what work is being done around them,
and it gives everybody a chance to receive feedback from other team members of
any important pieces of information they might have about the current work
being performed.

All three software teams work on feature branches based on a ticket from the
ticketing system. These branches are pushed up to Github, and turned into
Pull Requests, which are then reviewed before merging in a master branch.

Pair programming is encouraged, and team members initiate pair programming on
a volunteer basis.

Nestled in heart of Portland's Pearl district, the Househappy office is close to
a large number of restaurants, bars, coffee shops, food carts, and a Whole Foods
grocery store. The interior is well-designed and comfortable. It includes three
lounge areas and a café-style area with small tables and a bar. The biggest
meeting room features a Nintendo Wii U which gets a frequent workout.
