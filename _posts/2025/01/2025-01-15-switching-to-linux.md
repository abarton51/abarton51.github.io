---
title: "Finally Switching to Linux"
excerpt: A blog post about *my* thoughts and experiences jumping ship from Windows to Linux.
permalink: /posts/2025/01/01/switching-to-linux.md/
author: "Austin B."
---

## Introduction

Recently, I switched from Windows to Linux. By switched, I mean I completely wiped Windows OS from my device and booted Linux onto it. I have been a Windows user my whole life. Partly because I grew up PC gaming and also because it's just something I didn't think about when I was younger, so I used what was around me. Recently, I got more and more frustrated with the lack of control, the constant developer headaches, and an OS that felt like it was working against me more often than for me. After a semester of school had ended, I took some time to backup of files, flash a bootable USB drive, and wiped my machine clean.

What followed was a journey of discovery that not only rekindled my love for technology but also gave me a newfound appreciation for open-source software and the incredible community behind it. In this post, I want to share my thoughts on this step, some of the things I’ve learned along the way, and why I using Linux can go further than just a piece of technology. For many, this doesn't seem like a big step at all, and millions of others for decades now have done the same, so one would assume this really isn't a profound change at all. After all, it's just an operating system that you are changing. But for me, as someone that grew up their entire life using Windows, even though the change is quite simple and fundamentally not that large, it has resulted in drastic changes and improvements.

As a disclaimer, I'm not claiming what is the best operating system at all. I'm just discussing _my_ thoughts and findings.

---

## The Journey Begins

When I first booted into Linux Mint, I was greeted with an interface that was both familiar and refreshingly simple. It didn’t take long before I was knee-deep in learning all the things I could do. Here are just a few highlights:

- **Bash Scripting:** I discovered the power of the terminal and started automating repetitive tasks with bash scripts. This level of customization and control is really fun. Install scripts for dotfiles, cron jobs, sripts for verifying files, etc. 
- **Cron Jobs:** I started to automate simple tasks with cron jobs and bash scripts combined.
- **Customizing My Desktop Environment:** Customizing my desktop environment became a creative outlet. I made my workspace uniquely mine and learned a lot about desktop environments in general.
- **Neovim and Plugins:** Transitioning to Neovim for coding was a game-changer. My workflow has never been smoother and I feel like a have a much deeper connection to my source code.
- **Tmux Mastery:** Learning tmux added another layer of efficiency to my development process. It makes multitasking within the terminal seamless and along with Neovim it is unparalleled by any standard text editor.
- **DIY Projects:** I even started doing DIY things like setting up a DNS resolver on a Raspberry Pi. Tasks like this are so simple, there is tons of support, and it's enjoyable. In addition, when I SSH into a machine with an OS like Raspberry Pi OS, it doesn't feel foreign to me, it's just like my daily driver.

Every step of the way, I felt like I was peeling back layers of abstraction that had previously kept me in the dark. Linux made me feel like I was finally in control of my devices.

---

## Why Linux Mint

If you're not familiar, Linux is really just the kernel :neckbeard: :point_up:. The entire operating system is a combination of GNU software and the Linux kernel. A Linux *distribution* is just a variant of the GNU+Linux operating system. Each variant has its own advantages, and many variants are forks of a common one. For example, Ubuntu, Debian, Arch, and Linux Mint are all Linux distributions. Debian is a root distribution. That means, it was created from scratch from the GNU+Linux software. Ubuntu and Linux Mint are both derived from Debian. However, Ubuntu is a direct fork of Debian whereas Linux Mint is a fork of Ubuntu. ON the other hand, Arch is also a root distribution.

At the end of they day, it really doesn't matter all that much what distribution you use, and a lot of distributions are going to be and fell very similar. Generally, Ubuntu and Linux Mint are quite "beginner friendly" whereas Arch takes a bit more to use. Even so, they are largely the same.

I chose Linux Mint because of the prolific `apt` and `apt-get` package managers, Ubuntu has a ton of support available which also apply to Linux Mint, and it is a more lightweight and simple distribution in comparison to Ubuntu. I think this would be the perfect first Linux distribution for a lot of people, and it is a good step before switching to something like Arch (as I plan to do).

---

## Philosophies That Changed Everything

One of the most profound aspects of my switch to Linux was discovering and experiencing the philosophies that underpin it. It's a completely different thing to actually participate and use the technolgies under FOSS than it is to just talk about it. Oftentimes, people discuss a lot surrounding FOSS software, but don't actually use these technologies everyday and opt for proprietary and/or closed source. This disconnect really affects how your persepctive of these technologies and the philosophies that are rooted in them. Here is some of what makes Linux and the broader FOSS (Free and Open Source Software) community so powerful, but keep in mind that words are only so powerful, you should apply these philosphies:

### 1. **Freedom and Control**
On Linux, *you* decide what your system looks like and how it behaves. Unlike proprietary operating systems, Linux doesn’t dictate how you use your hardware. Want to strip down your system to just the essentials? Go ahead. Want to build a completely custom workflow? The tools are there.

### 2. **Transparency**
Open-source software means you’re never left guessing about what’s going on under the hood. You can inspect the code, see how it works, and even modify it to suit your needs. This transparency builds trust and fosters a deeper understanding of the technology we rely on.

### 3. **Community and Collaboration**
Linux isn’t just software—it’s a community. The level of collaboration and knowledge-sharing I’ve encountered is unmatched. From forums to GitHub repositories, there’s always someone willing to help or share insights.

### 4. **Open Source**
The open-source philosophy extends beyond just software. It’s about accessibility, learning, and the belief that technology should serve everyone, not just those who can pay for it. It’s empowering to be part of a movement that prioritizes user freedom over profit.

---

## How Linux Improved My Life

Switching to Linux hasn’t just been about improving my development workflow (although it’s done that in spades). It’s also fundamentally changed how I view technology and my relationship with it.

- **Greater Appreciation for Hardware:** Understanding how my system works at a low level has made me more mindful of how I use my devices and the resources they consume.
- **Love for Learning:** Linux has reignited my curiosity. Every new tool or project feels like an opportunity to grow.
- **Empowerment:** Knowing that I’m in full control of my system is incredibly liberating. No more bloatware. No more forced updates. Just a system that works the way *I* want it to.

---

## Final Thoughts

Switching to Linux has been one of the most rewarding decisions I’ve made in my pursuit of learning. It’s not just about using a different operating system—it’s about exploring philosophies of freedom, transparency, and community. Linux has made me a better developerr and, frankly, a happier person.

If you’ve ever thought about making the switch, I can’t recommend it enough. It might feel intimidating at first, but the journey is worth it. You’ll not only gain a deeper understanding of technology but also join a global community that’s working together to create something truly special.

If you're in a position similar to where I was, where you're quite familiar with Linux but from a guest machine perspective (virtual machines, containers, SSH, etc.), the transition really is smooth and you'll find it actually so much easier to use very quickly. Even if you do work with Linux a lot, but not as a daily driver, there will always be a disconnect between you and the system as well as the entire FOSS community.

Thanks for reading!

