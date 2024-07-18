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
Software Engineer professionally passionate about how we build software, both as individuals and as organizations. Serious about learning and get excited about the lessons that can be learned from failure. 

---
## Education
#### [Pennsylvania State University](https://bulletins.psu.edu/undergraduate/colleges/engineering/computer-engineering-bs/)
*Bachelor of Science in Computer Engineering*
University Park, PA
August 2019 -- December 2022
- GPA: **3.85**/4
	- Extracurricular learning materials that influence my thinking the most include Rich Hickey's numerous keynotes, Gene Kim's *Idealcast*, and Scott McConnell's *Code Complete 2*.

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