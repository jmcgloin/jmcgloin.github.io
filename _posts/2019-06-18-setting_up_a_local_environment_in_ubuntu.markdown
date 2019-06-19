---
layout: post
title:      "Setting Up a Local Environment in Ubuntu"
date:       2019-06-19 01:23:21 +0000
permalink:  setting_up_a_local_environment_in_ubuntu
---

While I understand the point to the  in-browser IDE and the use of a virtual machine when setting up a local environment when not on a Mac, I haven't really enjoyed the IDE and I'm not big on needing to set up a VM when I already have a Linux distro on my machine.  I've looked through the available instructions for setting up a local environment and, aside from the virtual machine, there's no guide to getting set up on Linux.  So, I set out to figure out how to do it.

[This post](https://help.learn.co/technical-support/local-environment/when-should-i-set-up-a-local-environment) has recommendations about when to switch to a local environment and links to instructions on how to set them up.  The instructions I detail here are based on the Mac instructions as the process for Linux is similar.  Note, I have only been able to test these instructions on my machine/distro and I cannot guarantee they will work for you.  If nothing else, they should be a good base to start from.  With a little independent Googling, you find out what you need to make it work.  Assuming I'm still around at the time you're reading this, feel free to hit me on Slack if you have questions.

Just to be clear, anywhere you  see a username or an email address, please replace it with your own.

## Step 0 - Start Clean
These instructions assume you are starting from scratch on your local machine but with a github account.  If you already have some apps installed or have certain preferences, you'll have to adjust the steps and maybe arrive at your own fixes.  If at anytime you're not sure about what installing something will do, you can always use `apt-get -s install <package_name>`.  The "-s" is the simulate flag and will allow you to see what would happen during the install without risking actually making the change until you're ready.
## Step 1 - Install git
There will be a few `apt update` in these instructions.  You really only have to do them (afaik) when you connect a new repository or if it's been a while.  It won't hurt (other than time) to do them all, but they're probably not all needed.

Enter the following in your terminal:
```
sudo apt-get update
sudo apt-get install git
```
This will install git locally, no connection to github or any repo host yet.  You can verify the installation using `git version`.

## Step 2 - Set up git
First, set up your username (local, not github)

```
git config --global user.name "MyDogDigger"
```
Then, set your commit email address.  You can use the no-reply email you get from github or any email you want to use.
```
git config --global user.email "digger@goodboy.com"
```
You can confirm these were set correctly using:
```
git config --global user.name
git config --global user.email
```

## Step 3 - Authenticating
In this step, you get local git  and github talking to each other so your code is saved and shared on github.  I cover the ssh method as it's what I've found I prefer.  You can also use HTTPS, which you can find instruction on [in github's help articles](https://help.github.com/en/articles/which-remote-url-should-i-use#cloning-with-https-urls-recommended).

Enter in your terminal:
```
ssh-keygen -t rsa -b 4096 -C "digger@goodboy.com"
```
You'll be prompted to enter a location to save the key.  Note the location, you'll need to navigate there soon.  The default location is fine.  Then you'll be prompted to enter a passphrase.  Do so or press enter not to use a passphrase.

Next, navigate to the location you were just prompted for.  For me it was in ~/.ssh .  Using your favorite editor, open id_rsa.pub.  Be sure it is ".pub", only share the public key, never the private key of the pair.  Copy the contents of the file (all of it).  Navigate to your github account an go to  Settings>SSH and GPG Keys.  Click "New SSH Key".  Give it a name, e.g. LapDogTopComputer, and paste the contents of the id_rsa.pub file in the "Key" text area.  To verify the connection, you can use `ssh -T git@github.com`.  You may see a warning about the authenticity of the host and a "fingerprint" will be displayed.  Make sure the fingerprint matches what's in github and type 'yes'.  You should then get a success message.  If not, you can troubleshoot using github's help articles, [like the article here.](https://help.github.com/en/articles/testing-your-ssh-connection)

## Step 4 - Support Libraries
In this step, you will install a couple support libraries.  gnupg was already installed on my distro and may be unnecessary for you to install.  The steps are
```
sudo apt-get update
sudo apt-get install libgmp-dev
sudo apt-get install gnupg
```
More info about these libraries can be found at [GMP](https://gmplib.org/) and [GNUPG](https://gnupg.org/).

## Step 5 - Install Ruby
If you need any help for this step, [this site was extremely helpful to me.](https://gorails.com/setup/ubuntu/18.04)  In fact, I will be using that site for the instructions of this step.  Start with the section titled "Installing Ruby", select the version of Ruby you want to use and essentially, copy and paste the commands.  The dependencies step will also take care of installing NodeJS which is a later step in the Mac OSX instructions.  Learn suggests using RVM for installing Ruby.  I tried this method first and ran into some issues with the  selected version of Ruby not actually being the version used.  This lead to many errors, much frustration, and lots of Googling.  In the end, using rbenv worked fine.  I cannot speak to long term issues this choice may present, so your mileage may vary.  Research until you feel comfortable with your choice.  Finish the section's instructions and, if you use rbenv,  don't forget to run `rbenv rehash`.

## Step 6 - Install Gems
Gems are little chunks of code that make your life easier.  Instead of needing to run several additional commands to get the desired behavior, the Gems take care of it behind  the scenes.  In this step, you'll install five gems.  First, start by updating the system gems
```
gem update --system
```
Now install the Learn gem, nokogiri, bundler, rails, and pry.

```
gem install learn-co
gem install nokogiri
gem install bundler
gem install rails
gem install pry
```
Again, if you went with rbenv, run `rbenv rehash`.

## Step 7 - Set up the Learn Gem
To  set up the Learn gem, enter the following `learn whoami`.  You'll get a prompt which will ask you to go to learn.co/your-githug-username.  You can also access this screen when logged into Learn by going to your icon menu and selecting "Your Profile".  Which ever method you use, go to the bottom of the page and copy the OAuth token (just the stuff after the ":").  Paste this into your terminal and press enter.

## Step 8 - Learn Configure
I am assuming that you already have an editor chosen.  If not, you'll need one going forward, now's a good time to get one.  I personally use Sublime Text, Learn recommends VS Code, and some fellow cohort-mates and tech advisers use Atom.  Read up on them, get some input from others, and make your choice.  Once you have an editor, open `~/.learn-config`.  You should have fields for learn_directory and editor.  Change the values to reflect the location you want to use for your learn files and your editor of choice.

## Step 9 - Databases
Now it's time to install SQLite and PostgreSQL.  SQLite is  quite straight forward:
```
sudo apt-get install sqlite
```
For PostgreSQL, go to [the Ubuntu download page of their site](https://www.postgresql.org/download/linux/ubuntu/).  Select your Ubuntu version and  follow the instructions.  When creating and modifying the file, you may need to set the chmod permission or use sudo.

## Step 10 - Chrome
If you've even made it to this blog post you must have a browser.  If you don't have Chrome, get it.  You'll use the developer tools of chrome quite a lot and there are some pretty nice extensions to use for web dev work. [In case you don't know where to find it or like clicking more than typing](https://www.google.com/chrome/).

## Step 11 - DB Browser for SQLite
DB Browser is a visual tool for working with databases.  They, like hashes for instance, can get pretty tough to read.  This tool gives a different way to view databases that may be easier.  To install, add the PPA, update apt-get, and install it.
```
sudo add-apt-repository -y ppa:linuxgndu/sqlitebrowser
sudo apt-get update
sudo apt-get install sqlitebrowser
```

## Done!
At this step you should be ready to go.  As I've said, these instructions worked for my setup.  I cannot guarantee they will work as is for you, but they should at least be a good reference.  You may need to do a bit of googling for your specific setup.  Also be sure to check the links I've provided, especially if you're attempting this months or years after I've written it (5/2019).  Tech tends to change quickly and there may already be some important differences.  If I'm still around, contact me via Slack if you have any questions or feedback.  I'll help as I can or at least update this blog post to reflect your findings/issues.

Thanks for reading.  I wish you the best of luck!
