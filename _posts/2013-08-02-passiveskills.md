---
title: Passive Skills
layout: post
---

One thing that I've been intentionally doing for a long time but especially this summer is working on my supporting skills. Certainly the actual skill of writing code is most important in being a programmer, as that's the main activity, but one's productivity and speed can be increased by correctly maintaining one's environment and by enriching skills that allow one to focus more on writing code and less on one's environment.

This has taken form for me most obviously as my dotfiles repository on GitHub, where I keep my configuration files so that my environment can be quickly duplicated over multiple machines or in the case of a hardware error, but it always boils down to removing repetition in my workflow wherever it appears. Namely, the ability to strip out repetition has been a skill in itself that I've noticed has improved over the last few months.

This afternoon I realized that my dotfiles repository, although it contains the folders for my vim plugins in the `bundle/` folder, was not tracking those files because they're all their own git repositories, since I install vim plugins using Pathogen. This meant that I needed to set up modules for about twenty git repositories. I saw this potentially tedious task as an opportunity to improve my Bash skills.

For each file, I needed to find the url in the `.git/config` file in each repo, disregard any singleton files or folders that are not git repositories, take the name of the file and the pathname to the submodule, and add all of this information in the specified format to .gitmodules.

I spent a few minutes and whipped up a Bash one-liner to do this for me.
I executed it as a single command but I've formatted it nicely and removed my prompt for clarity here:

{% highlight bash %}
for x in `ls vim/bundle`; do 
  if [ -d vim/bundle/$x/.git ]; then 
    url=`cat vim/bundle/$x/.git/config | grep url | awk '{print $3}'`;
    echo -e "\n[submodule \"vim/bundle/$x\"]\npath = vim/bundle/$x\nurl = $url" >> .gitmodules; 
  fi 
done;
{% endhighlight %}

In short, this lists everything in Pathogen's `bundle` directory, then iterates through them. I then check with `-d` to be sure that the item is a directory containing a `.git` directory. If it is, I use `grep` and `awk` to pull the URL of the remote repository from the `.git/config` file and save that as `url`. Then we generate the git submodule format and concatenate it to `.gitmodules`.

It's quick, it's dirty, it's a one-off script, but I was able to save myself some time (which I promptly used to write this) and some tedium and got a little better with Bash while I was at it. Any time I get better with my shell (or any part of my environment, which was the point of this post), the faster I'll be at solving these kinds of intermediary problems and at shaving yaks during projects going forth.
