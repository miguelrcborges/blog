---
  title: Tried out wayland. It wasn't the best experience.
  date: Mon Jun 13 05:06:35 PM WEST 2022
  keywords: linux, graphical server
---

# So what made me try wayland

I like the idea of integrated compositor with the window manager. Not only that, Wayland is visually smoother.
However, in my two days experience, I came across a lot of problems.

## The problems

* Screen Artifacts. This was a more relevant problem on the fullscreen mode.
  * This problem is most likely caused by using nvidia on wayland, sadly.

* Each compositor has to implement his own calls. This is not so bad if every compositor turns to be wlroots based.

* Setting up xdg-portal isn't easy, or maybe I was unlucky with some incompability.



# Conclusion

That being said, I won't use wayland again on my desktop most likely soon. When it turns more mature, I'll give it another try.


EDIT: Seems the problems I was having with xdg-portal were related to the compositor.
