---
title: "Removing git branches in batch"
layout: post
date: 2016-06-04 04:38:13
image: '/assets/images/'
description:
tags:
blog: true
jemoji:
---

Sometimes, we get a point where we have many branch that we don't need anymore. It can be a tedious task delete them one by one. 

In order to avoid that, I made an script where we just need to specify which branch we don't want to delete and that it's. Just run the script and that is all, your ugly/unfriendly branches have been gone.

{% highlight bash %}

#!/bin/sh
# branch list to not delete
branch_not_delete=( "master" "develop" "our-branch-1" "our-branch-1")


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