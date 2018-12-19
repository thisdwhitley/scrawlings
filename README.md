I am currently doing testing using a hugo container because I don't want to install all the required packages.  That looks like:

~~~
docker run --rm -it -v $(pwd):/src -p 1313:1313 klakegg/hugo:0.52 server
~~~
