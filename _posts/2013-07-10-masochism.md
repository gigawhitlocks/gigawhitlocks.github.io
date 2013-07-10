---
title: Dotfiles and Masochism
layout: post
---

Yeah, today I decided to move all of my dotfiles into a proper git repo, and replace them in their normal locations with symlinks to the organized folder on my system, for easy replication of my vim/i3/etc environments. I was feeling masochistic, so I wrote [a Bash script](https://github.com/thewhitlockian/dotfiles/blob/master/migrate.bash)to simplify the process of adding new dotfiles to my system and of installing my existing dotfiles onto a new system. I particularly enjoy this recursive backup function that I wrote to be sure nothing that shouldn't be deleted is deleted:
{% highlight bash %}
		function backup {
				# recursively generates filename.bak backups
				# on file as first arg

				if [ -e $1.bak ]; then
						backup $1.bak;
				fi
				echo "mv $1 $1.bak";
				mv $1 $1.bak;
		}
{% endhighlight %}
