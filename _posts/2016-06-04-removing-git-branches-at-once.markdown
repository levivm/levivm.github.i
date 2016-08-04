---
title: "Removing git branches at once"
layout: post
date: 2016-06-04 04:38:13
image: 'http://nvie.com/img/merge-without-ff@2x.png'
description: How to remove several branches at once, locally and remotly.
tag: GIT
tags:
- git
- branches
blog: true
jemoji: ':octocat:'
---

Sometimes, we realized that we have many branches that we don’t need anymore. It can be a tedious task delete them one by one. 

In order to avoid that, we can automate this task. Using this script, you just need to specify which branches you want to preserve, run it and that is all. Your ugly/unfriendly branches should been gone.

{% highlight bash %}

#!/bin/sh
# branch list to not delete
branch_not_delete=( "master" "develop" "our-branch-1" "our-branch-2")


# Iterate over remotes branch and if they aren't in our previous list, we deleted # them
for branch in `git branch -a | grep remotes | grep -v HEAD | grep -v master`; do

	# delete prefix remotes/origin/ from branch name
	branch_name="$(awk '{gsub("remotes/origin/", "");print}' <<< $branch)"
	
	if ! [[ " ${branch_not_delete[*]} " == *" $branch_name "* ]]; then
		# delete branch remotly and locally
    	git push origin :$branch_name
	fi
done 
{% endhighlight %}