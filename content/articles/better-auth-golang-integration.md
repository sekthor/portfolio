+++
title = 'Better Auth Golang Integration'
date = 2026-03-13T21:33:51+01:00
tags = []
technologies = []
draft = false
+++

I am currently working on a virtual tabletop game.
The working tile is *TaleVTT*.

It consists of a NextJS frontend and a backend of golang micro services.
For authentication I am using Better Auth.

Users are part of a campaign.
Users should be able to see other users of mutual campaigns.
The issue I was facing was displaying user information to other authenicated users.
Better Auth supports Organizations that allow users to see their fellow organization members.
However, I do not want to rely on that mechanism to much to shield myself from a possible vendor-lockin down the line.

This means rather than storing user information redundantly in my own backend, I needed a way for golang to fetch this info directly from the identity provider (Better Auth).
As I did not find a golang library that already supports this, I started my own.

The project is publicly available at [github.com/sekthor/go-better-auth](https://github.com/sekthor/go-better-auth).
So far it supports user creation, user signing (both using email/password) and fetching user details using the admin api.
I am planning to expand the package to support most of the standard API and the plugins I am currently using.

I have already integrated the client in my TaleVTT PoC.
What I have not yet solved is how I am going to handle token expiration.
I will need to find a way to refresh the token or implement long lived access (e.g. API Tokens).

In addition, bootstrapping the application on initial install is becoming more tricky.
Right now I can create accounts using my TaleVTT CLI, using the Better Auth API.
But I still need to manually set the account's role, to allow it to list users.
This leaves the problem of implementing a restricted role (rather than full admin privileges) and creating service accounts with that role without manual intervention for future me.
