---
layout: post
title: Sample blog post
subtitle: Each post also has a subtitle
gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [test]
comments: true
---

This is a demo post to show you how to write blog posts with markdown.  I strongly encourage you to [take 5 minutes to learn how to write in markdown](https://markdowntutorial.com/) - it'll teach you how to transform regular text into bold/italics/headings/tables/etc.

**Here is some bold text**

## Here is a secondary heading

Here's a useless table:

| Number | Next number | Previous number |
| :------ |:--- | :--- |
| Five | Six | Four |
| Ten | Eleven | Nine |
| Seven | Eight | Six |
| Two | Three | One |


How about a yummy crepe?

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg)

It can also be centered!

![Crepe](https://s3-media3.fl.yelpcdn.com/bphoto/cQ1Yoa75m2yUFFbY2xwuqw/348s.jpg){: .mx-auto.d-block :}

Here's a code chunk:

~~~
var foo = function(x) {
  return(x + 5);
}
foo(3)
~~~

And here is the same code with syntax highlighting:

```javascript
var foo = function(x) {
  return(x + 5);
}
foo(3)
```

And here is the same code yet again but with line numbers:

{% highlight m68k %}
;	bsr LSP_MusicDriver+4 : LSP player tick (call once per frame)
;		In:	a6: should be $dff0a0
;			Used regs: d0/d1/a0/a1/a2/a3/a4/a5
;		Out:None
;
;*****************************************************************

;	opt o-		; switch off ALL optimizations (we don't want vasm to change some code size, and all optimizations are done!)
	
LSP_MusicDriver:
			dc.w	$6000
.rel:		dc.w	.LSP_PlayerInit-.rel

;.LSP_MusicDriver+4:						; player tick handle ( call this at music player rate )
			lea		.LSPVars(pc),a2
			move.w	m_lastDmacon(a2),d0
			beq.s	.skip
			lea		m_resetv(a2),a3
			lea		16*4(a6),a4
			moveq	#4-1,d1
.rLoop:		lea		-16(a4),a4
			btst	d1,d0
			beq.s	.norst
			move.l	(a3)+,(a4)
			move.w	(a3)+,4(a4)
.norst:		dbf		d1,.rLoop

.skip:		lea		m_streams(a2),a1
			moveq	#4-1,d7
			moveq	#0,d6
			lea		m_resetv(a2),a3
			lea		16*4(a6),a6
			
.vLoop:		lea		-16(a6),a6
			move.l	(a1),a0
			move.b	(a0)+,d0		; cmd for current voice
			move.l	a0,(a1)			; update cmd stream ptr
			add.b	d0,d0
{% endhighlight %}

## Boxes
You can add notification, warning and error boxes like this:

### Notification

{: .box-note}
**Note:** This is a notification box.

### Warning

{: .box-warning}
**Warning:** This is a warning box.

### Error

{: .box-error}
**Error:** This is an error box.
