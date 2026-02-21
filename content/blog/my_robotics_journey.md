+++
date = '2026-02-21T10:02:04Z'
draft = false
title = 'My Robotics Journey'
+++

I wrote this many months ago, but never posted it. I think it's worth seeing the light of day. The conclusion section is new, however. The topics I discuss, my robot arm and quadcopter, are finished and partially-finished-then-abandoned respectively. Eventually I'll finish off the quadcopter post... one day.

---

This should act as a recollection of my journey to get to where I am now, but also as somewhat of a guide for getting into building hobbyist robots. In fact, I think I'll split this write-up in half, and deal with each topic in turn.

# My journey

For context, as of the time of writing I have built a small servo motor powered robot arm that I designed, 3D printed, wired up (not hard since it was just connecting servo motors to a Raspberry Pi Pico W), and _somewhat_ programmed. I am part way through building my own quadcopter from scratch, including designing the PCB and programming the flight computer (a **much** harder project). 

## My background

### Maths
I'm in my third-year of my university course, although I'm currently on a placement-year, working as an Operational Researcher for the Civil Service. My degree is in Mathematics and Statistics, which is just a maths course, but I've picked a lot of statistics modules. My degree has been fundamental and instrumental to the development of my technical skills, mostly by developing the ways I think about problems and structure. If that all sounds very abstract, it's because it is - but I cannot understate how helpful the ideas presented in courses like linear and abstract algebra have been to helping me become a better programmer. Interestingly, I loved linear algebra and abhorred abstract algebra - linear algebra felt very grounded in geometry, whereas abstract algebra felt, well, *too abstract*, and quite number theory heavy - especially the chapters on polynomials within ring theory...

### Programming
While I've always been interested in becoming a programmer, I can't say I ever actually created a substantial project until very recently, with my arduino Neovim plugin. My first brush with programming was learning scratch in primary school when I was 9. I then learnt bits of Python from a book my parents bought me, and learnt some more during highschool, but I must reiterate, I wasn't actually _doing_ anything with any of this knowledge, because I hadn't found anything I actually wanted to apply my programming """skills""" (I was 14 years old before I could understand how the for loop syntax worked in python...) to. 
I studied computer science during my GCSEs and A-Levels and continued to learn in my spare time (still never really making any substantial projects). I learnt more Python, some HTML, CSS, and Javascript (almost all of which I have forgotten), we learnt Visual Basic during our A-Levels (chosen for its easy-to-read syntax and the easy-to-use GUI editor within Visual Studio, although, I do wish they had chosen a language that people actually _used_), and I even learnt a bit of C, which I liked quite a lot. I learnt C while learning some algorithms and data structures (I barely made it past linked lists before neglecting to continue my studies) from the book ==book name goes here==. At university I learnt R and in my own time I learnt Python's numpy and pandas libraries, as well as some other data science related stuffs. For data science, I prefer R, personally. 
In developing my recent robotics projects I have learnt a lot more embedded C, mainly for use with the Arduino libraries. I also learnt some FreeRTOS to go along with it, which was really nice because I got to see how all the stuff I learnt back in my A-Level computer science operating systems lessons worked in the real work: schedulers, interrupts vs polling, buffers, etc., I've got to see these all in action while working on my flight computer.

### Electronics

At the start of my two years at college (aged 16), I bought my first Arduino Uno, beginning a long foray into the world of electronics. I learnt lots via tutorials, but as with programming, I failed to find any projects that I was really interested in to apply myself to for a long time.
My first real successful project was designing Th!nk, my second wireless-split-ergonomic-ortholinear-column-staggered-super-weird-looking keyboard, powered by ZMK. 
I designed Th!nk myself, following along with videos from Ben Vallack, who anybody who has built a keyboard like this will be familiar with, and other sources. The PCB design, in hindsight, was horrendous, but despite that (and needing two bodge wires), Th!nk worked, and I used it for the next two years. More recently I've switched over to a wireless corne, because I prefer the extra row for dedicated ctrl and shift keys (and ESC and print screen as a bonus).
More recently, I got into robotics, which led me to develop a robot arm, which isn't very interesting from an electronics stand point, it's just 6 servo motors hooked up to a Raspberry Pi Pico W. What is way more interesting is my quadcopter, for which I designed the PCB myself, and did a *much* better job on than with Th!nk. I learnt pretty much everything from Phil's Lab on YouTube, the Electrical Engineering stack exchange, and reading the first couple chapters (and skimming other parts) of Practical Electronics for Inventors.

## Conclusion and a note on changing your mind

As I mentioned at the start, I wrote all of that many months ago. I've now decided that while I love working on robotics, I want to do it in my spare time for now. Trying to break into the engineering companies around here was just too much effort for too little gain - I'd rather a more flexible work life balance working in a data science role, where I can simultaneously continue my statistical studies (which I've recently been enjoying much more) and work on the projects *I want to work on* in my spare time, using the tools *I want to use*. That's really important to me - I spoke to somebody who worked at an engineering company, and he told me he used Excel to generate samples from a normal distribution or something like that. God forbid a technical business in 2026 have access to R or Python... I'm much happier having spare time to work on projects I care about using likeable technologies, than working on something slightly adjacent from my true interests in Excel (🤮).

The biggest take away from my journey is that I change my mind all the time, and that's OK. As I was discussing with one of my flatmates yesterday, it's not uncommon to hear about people who have been doing one thing their entire life and have gotten really good at it, and imagine that following their path is the only way to success - that if you say "maybe I don't want this, maybe I should change course", you're setting yourself back and will never reach the tippity-top of some conjured up societal foodchain, but this isn't true. Plenty of people explore before they settle down then find great success.

It's too easy to be a young 20-something like myself and think "why hasn't my story started yet!" - but not everyone is meant to be the young hero. I myself want to be the wise old wizard, so I guess my story starts when I'm like 70. If that mental model doesn't do it for you, go recount that old story about Caesar looking at Alexander's statue and lamenting how he hasn't done anything with his life. 

We have time, and slowly but surely we move forwards.
