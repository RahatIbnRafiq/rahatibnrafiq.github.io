---
layout: archive
title: "Flutter Cocoapods Ruby Mess"
permalink: /flutter_ruby_cocoapods_mess/
author_profile: true
---

# Background

I have been working on the wwm app for a month now. Everything has been going on seamlessly, the development, althought not super fast, was brisk. I have been making a pull request almost every week. Until last week; Saturday. I woke up in the morning, went to the coffee shop with Nammi to work, tried to pull up the ios simulator to run the app and well, nothing comes up! The issue was: **You seem to have cocoapods installed but not working.**

# Fix

Let's just say I spent **a lot of time** fixing what was indeed the issue. After numerous links, stackoverflow, giothib issues, reddit; here was the issue:

1. MacOS Sonoma uses ruby 2.6, which is something you really do not wanna touch
2. New Flutter versions and all that needs new versions of ruby and cocoapods. Newest stable version of Ruby is 3.0.0
3. You have to make sure your default version of ruby is 3.0.0 or whatever you want to upgrade it, not the default one that Mac's using
4. Use rvm to install the new version of ruby, make it default. Otherwise your project will always go back to using the 2.6.whatever mac's using. Make sure to install rvm (or any other ruby version manager). Also make sure to install with your openssl. **which openssl** should give you the folder for openssl. Use that to **rvm install 3.0.0 --with-openssl-dir=$dir-to-openssl**
5. Then install cocapods for that ruby version
6. Flutter doctor should work after that.

## Another issue

If after all that, you are getting the following errors:
1. flutter Unable to find a target named `RunnerTests` in project `Runner.xcodeproj`, did find `Runner`.
2. Flutter Automatically assigning platform `iOS` with version `12.0` on target `Runner` because no platform was specified. Please specify a platform for this target in your Podfile. See `https://guides.cocoapods.org/syntax/podfile.html#platform`

# Fix

In the ios folder, delete the podfile, the pod lock file. Then rerun. If the issue persists, then do the following:

1. uncomment the ios version line in the podfile
2. uncomment these lines in the podfile:   
```
target 'RunnerTests' do
  inherit! :search_paths
end
```
That should do the trick!

