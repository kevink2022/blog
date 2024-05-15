---
title: Resume
menu: main
weight: 5
---
# Resume
[PDF Download](/blog/docs/resume.pdf)

---
# Kevin Kelly
[kevink2019@gmail.com](mailto:kevink2019@gmail.com) | [GitHub](https://github.com/kevink2022) | [LinkedIn](https://www.linkedin.com/in/kevink2019/)

---
## Education
#### [Pennsylvania State University](https://bulletins.psu.edu/undergraduate/colleges/engineering/computer-engineering-bs/)
*Bachelor of Science in Computer Engineering*
University Park, PA
August 2019 -- December 2022
- GPA: **3.85**/4

---
## Experience
#### [FAST Enterprises](https://www.fastenterprises.com/)
*Implementation Consultant*
Sacramento, CA
July 2022 -- Present
- Effectively communicate with clients to understand their business requests, discuss requests to ensure they address root business problems, and created tailored solutions within the FAST system, adhering to **object oriented** and internal best practices.
- Develop and refactor **SQL** queries with high standards of readability, maintainability, and performance. Ensure SQL query performance through Execution Plan and cardinality statistics analysis.
- Deliver confident and impactful client demonstrations and technical presentations, effectively communicating complex technical concepts to audiences of varying levels of technical expertise.
- Initiate discussions about and tirelessly advocate for stronger software oriented organizational principles. Examples include adopting style guides for the languages we use, adding unit and integration testing to our testing procedures, and refactoring for future maintainability while making changes.
#### [RTD Embedded Technologies](https://www.rtd.com/)
*Engineering Intern*
State College, PA
May 2022 -- August 2022
- Maintained and updated Legacy **Linux Kernel Drivers**, adeptly diagnosing issues and analyzing Linux Kernel Source code to resolve bugs and implement driver updates.
- Developed a **Python** tool for automated driver retrieval and testing from repositories, effectively identifying and addressing widespread bugs resulting from Kernel updates.
- Diligently adhered to the comprehensive document control protocols required by ISO 9001 and AS9100 standards.
- Successfully ported legacy ROM DOS manufacturing tests to Python scripts for Linux, modernizing the PCB production testing process. 

---
## Projects
#### [Boomic Music](https://github.com/kevink2022/Boomic)
*Swift, SwiftUI, Functional Programming, Persistent Data Structures, Combine, async/await*
January 2024 -- Present
- iOS music playback app for high resolution audio codecs (e.g. FLAC) and music library manager.
- Automatically populates the music library through the iOS Files app, allowing for drag and drop. Organizes music into albums and artists based on metadata and displays associated album art.
- Built with **persistent data structures** and **functional programming principles** while still taking advantage of SwiftUI's powerful observation libraries using **Combine queues** for efficient, decoupled and reactive communication.
- Utilizes a transaction based persistence system, allowing for users to roll back erroneous changes.
- Thorough **unit testing**, often following **test driven design**, writing tests before the code.
- Architecture and design decisions discussed in detail on my blog at [kevink2022.github.io/blog/tags/boomic/](/blog/tags/boomic/)
#### [Channels](https://github.com/kevink2022/channels)
*C, GDB, Valgrind, pthread, semaphores*
January 2022 -- May 2022
- Multi-thread safe buffered channel system enabling threads to send or receive data. 
- Incorporates a 'select' function enabling threads to submit multiple send/receive operations, ensuring that only one operation is completed.
- Implemented in **C**. Employed linked lists for queue management, **mutex locks** for thread safety, and **semaphores** to control access to resources. 
- Utilized **GDB** and **Valgrind** for debugging and memory safety.

---
# What I am looking for in an organization
I understand that in an ideal world, all organizations would follow software development principles and best practices. However, we live in a unideal world of capital, competition, deadlines, and security vulnerabilities. These *external pressures* often constrict an organizations choices. Time spent preparing for the far future is time wasted if the organization might cease to exist in the near future.

However, most organizations that last will find themselves in periods where those external pressures subside, and then the *internal pressures* they control, conciously or not, drive the further direction of the organization. I firmly believe this is where organizations truly distinguish themselves, and by following strong principles, set themselves up for success. 

When the external pressures begin to ramp up, the organization that prepared in the past, learned from its previous mistakes, and made multiplicative process improvements, will have a lot more room to maneuver. Meanwhile, the organization ignores past mistakes and focuses entirely on the next venture, will find itself completely constricted by external pressures, leading to at best an intense period of stress, and at worse, the collapse of the organization.

I understand that external pressures can demand short-term thinking. But any professional relationship I enter with an organization that either explicitly encourages permanent short-term thinking, or even worse, doesn't understand the implicit effects of the internal pressures they unconsciously create, will not last.

Some of the ways I think mature software organizations are built to last are:
- Aggressive testing, where developers aren't [too reliant on QA](/blog/learning/better_testing_worse_quality).
- [Fast feedback loops](https://youtu.be/r2BFTXBundQ?t=503&si=QBm-rZdg97pn_Y2o) to avoid the pitfalls of rampant speculation.
- Introspective, learning from past mistakes, and is willing to face and fix problems instead of avoiding them.
- Proud of its most simple and readable code, and embarrassed by incidental complexity.
- Embraces functional design where appropriate and is wary of the dangers of shared state.