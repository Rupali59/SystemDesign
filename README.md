# System Design
Resources for reading about system design

* [ ] [Design Patterns](https://sourcemaking.com/)
* [ ] [System Design Interview Preperation Series by CodeKarle](https://www.youtube.com/watch?v=3loACSxowRU&list=PLhgw50vUymycJPN6ZbGTpVKAJ0cL4OEH3)
* [ ] [HiredInTech](https://www.hiredintech.com/classrooms/system-design/lesson/53)
* [ ] [system-design-primer](https://github.com/donnemartin/system-design-primer)
* [ ] [Google prep](https://github.com/mister0/How-to-prepare-for-google-interview-SWE-SRE)
* [ ] [Design-Microservices-Architecture](https://github.com/mehmetozkaya/Design-Microservices-Architecture-with-Patterns-Principles)
* [ ] [Karan Pratap Singh](https://www.karanpratapsingh.com/courses/system-design)
* [ ] [ByteByteGo](https://github.com/ByteByteGoHq/ml-bytebytego)

## Approach
* **Be absolutely sure you understand the problem being asked**, clarify on the onset rather than assuming anything 
* **Use-cases**. This is critical, you MUST know what is the system going to be used for, what is the scale it is going to be used for. Also, constraints like requests per second, requests types, data written per second, data read per second.
* Solve the problem for a **very small set**, say, 100 users. This will broadly help you figure out the data structures, components, abstract design of the overall model.
* Write down the various components figured out so far and how will they interact with each other.
*  As a rule of thumb remember at least these :
	 * processing and servers
	 * storage 
	 * caching 
	 * concurrency and communication
	 * security 
	 * load balancing and proxy 
	 * CDN 
	 *  Monetization: if relevant, how will you monetize?
 eg. What kind of DB (Is Postgres enough, if not why?), do you need caching and how much, is security a prime concern? 
* **Special cases** for the question asked. Say designing a system for storing thumbnails, will a file system be enough? What if you have to scale for facebook or google? Will a nosql based database work?
* After I have my components in place, what I generally try to do is look for minor optimization in various places according to the use-cases, various tradeoffs that will help in better scaling in 99% cases.
* [Scaling out or up](http://highscalability.com/blog/2014/5/12/4-architecture-issues-when-scaling-web-applications-bottlene.html)
* Check with the interviewer is there any other special case he is looking to solve? Also, it really helps if you know about the company you are interviewing with, what its architecture is, what will the interviewer have more interest in based on the company and what he works on? 
