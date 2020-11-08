When the pandemic hit, I scrambled to determine a way to work from home with my embedded system. The system was portable. I squeezed everything I needed into a large box. I already had extra monitors and other accessories in my home office. Hence working from home was viable.

Developing embedded software is a cycle of building the code, flashing the binary into the embedded target hardware, and testing it. And repeating this cycle until the team has all the features developed and bugs fixed.

Here are some valuable lessons I learned during this process.

<!-- TEASER_END -->

## Document The Developer Setup
Each developer will need to transport the system back and forth from work for integration testing. Document your development setup with diagrams to avoid hiccups each time you have to move the equipment. Share the setup with others using your official documentation repository (you have one, right?). Not email.

## Draw A Diagram Of Your Unique Configuration
Your home-based hardware setup is not visible to your colleagues. Hence troubleshooting can be challenging. When one developer used a 100Mbit Ethernet switch instead of the required Gigabit Ethernet switch, something failed. It took hours of remote discussion and troubleshooting to realize that minor difference. Sketch out a precise diagram of your setup as it may differ from the standard developer setup. This will save time when troubleshooting later.

## Get Unstuck Fast
A developer can get stuck on a problem and may noodle on the issue longer than needed before asking for help. Developers tend to be more introverted on average compared to the general population. Daily online meetings are more important than ever to keep the momentum going. Of course, get help from your colleagues without waiting for the meeting.

## Shutdown At The End Of The Day
It’s easy to keep working through the evenings and weekends when there’s a bug to fix and there’s no need to “leave work”. While difficult to follow, I found that sticking to a reasonable work schedule was more productive. Getting my mind off the problem for a few hours or overnight often gave me new insights that helped solve the problem with less effort.

## Keep Extra Equipment Around
I had collected a variety of Ethernet switches, USB adapters, USB hubs and cables over time. These were invaluable in facilitating two different embedded project setups. Recall that these were the initial weeks of the pandemic. Online stores stopped offering two-day shipping. In fact, there was no clear shipping timeline promised for anything.

## Create Project Onboarding Instructions
Even during ordinary times, some team members may get moved to other projects, and new team members will pop up. To ease onboarding, create documentation that covers the software build and load instructions, the project's Git repository management philosophy, and the hardware setup. Large teams couldn’t survive without such documentation. I've noticed that this is often neglected by small teams and impacts team productivity later.

## Some Limitations Of Working From Home
I found working from home more effective than the more distraction-filled cubicle-world. However, the team loses the valuable ad-hoc discussions that often sprout within a co-located team.

To be effective when working from home, one needs to have already made sufficient connections within the company. For someone new who has not yet established the connections, it will be more challenging.

For my recent embedded software projects, there was a limit to the amount of development and testing possible at home. At some point, the integration of the prototype with the rest of the system at work was necessary.
