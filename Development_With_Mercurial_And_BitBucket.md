# Introduction #

XMind is a really nice application and it's great that the source code is available so people can make tweaks or even add new features.

I think the current set-up can be improved by putting the source inside a version control system and making it easier to track and share patches. Thus, I've created a mirror of the released source code on BitBucket. For those who haven't used it, it's a very good webapp for hosting source-code that's based on the Mercurial version-control system with features for sharing and following changes.

# Details #

Here are my current repositories on BitBucket:
  * https://bitbucket.org/eug48/xmind3-releases-mirror
  * https://bitbucket.org/eug48/xmind3-external-deps
  * https://bitbucket.org/eug48/xmind3-patches (mq repository with a few of my tweaks)

# Building within Eclipse #

You probably want to be a little familiar with Mercurial - there are many online tutorials and it should be time well-spent :)
  1. Install Mercurial (e.g. by downloading TortoiseHg)
  1. Clone the Mercurial repositories
    1. Run `hg clone https://bitbucket.org/eug48/xmind3-releases-mirror`
    1. Run `hg clone https://bitbucket.org/eug48/xmind3-external-deps`
  1. Start Eclipse and select a workspace in a new _empty_ folder. I'm using Eclipse 3.7 (Indigo) **for RCP and RAP Developers**, with Sun JRE6.
  1. Import XMind's external dependencies
    1. Go to File --> Import
    1. Select "Plug-ins and Fragments"
    1. Choose to import from a directory and select the directory where the xmind3-external-deps repo has been cloned
    1. Choose to select from all plug-ins
    1. Choose "Binary projects with linked content"
    1. Press next
    1. There should be 3 plug-ins available. Press "Add all".
    1. Press finish. The three plug-ins should be imported without build errors.
  1. Import the XMind source code
    1. Go to File --> Import
    1. Select "Existing projects into workspace"
    1. Under "Select root directory" pick the directory where the xmind3 source (e.g. xmind3-releases-mirror) has been cloned
    1. Press "Select All"
    1. Press Finish.
    1. I'm currently getting 1 build error - "An API baseline has not been set for the current workspace." To fix it: Preferences --> API baselines --> Add Baseline --> type a name --> press Reset --> press Finish --> accept a full build. There should now be no errors.
  1. Run XMind from within Eclipse
    1. Go to Run Configurations
    1. Expand "Eclipse Application"
    1. You can select the pre-existing Windows/OSX or duplicate one of them to make a config for Linux. You may need to click "Add Required Plug-ins" in the Plug-ins tab..
    1. Press Run - XMind should start.

# Building stand-alone packages #
TODO

# Collaboration #
With BitBucket it's easy to fork the mirror to create your own repository. This will hopefully make it easier to get started and synchronise to new releases. They also let you pull code from others, "follow" other people's updates and more.


