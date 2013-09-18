---
title: Fizzbuzz in Racket
layout: post
---

This post is just the source for [Fizzbuzz](http://en.wikipedia.org/wiki/Fizzbuzz) in [Racket](http://racket-lang.org), which I wrote this morning for fun since I'm starting to get a handle on Racket and was pleased to use this iterative pattern which I found in the first chapter of SICP to implement this simple puzzle.

{% highlight racket %}
(define fizzbuzz
; this is a default-args wrapper
; which does the usual 1-100 run
  (lambda ()
    (fizzbuzz-iter 1 100)))

(define fizzbuzz-iter
  (lambda (iter max)

    ; the actual meat of fizzbuzz
    (cond [(and (= (remainder iter 5) 0) 
		(= (remainder iter 3) 0)) 
	   (printf "FizzBuzz\n")]
	  
	  [(= (remainder iter 5) 0) 
	   (printf "Buzz\n")]

	  [(= (remainder iter 3) 0) 
	   (printf "Fizz\n")]

	  [else (printf "~a\n" iter)])

    ; call the iterator
    (if (< iter max) 
	(fizzbuzz-iter (+ 1 iter) max) 
	(printf "\n"))))
{% endhighlight %}
