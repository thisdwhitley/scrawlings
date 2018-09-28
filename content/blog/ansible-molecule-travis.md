+++
title = "Testing Ansible with Molecule AND Travis"
date = "2018-09-26T08:52:19-04:00"
tags = ["ansible","molecule","travis","ci"]
categories = ["tech"]
banner = "img/banners/banner-2.jpg"
+++

At one point I was restarting all my ansible roles to include testing with
molecule.  Then I realized I could test with github and Travis-ci which was also
cool.  Then I finally realized that I can do both, but use the same tests.

Basically, this will look like:

1. build out a role locally
2. test locally in various OSes (in containers) locally with molecule
3. then push to github and perform the same tests once pushed (CI)

This is sort of based on https://werner-dijkerman.nl/2017/09/05/using-molecule-v2-to-test-ansible-roles/