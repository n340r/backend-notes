# ✨ Soft Skill Dumb Questions

This one is dedicated to **HR** / **Behavioral** / **Engineering Manager** questions.

**Engineering Manager** interview is a **1.5hr** long discussion of your experience with main focus on your responsibility and impact.

### (For me only) Speech disclaimer

“I want to mention that I have a stutter problem that can show up under stress. I’m currently **undergoing speech therapy**, so i might **speak slowlier than usual** or **take a moment to phrase something**, that’s expected. With that out of the way lets proceed”

## 🇬🇧 Basic

### Tell us about your most recent workplace and the project you worked on.

---

I work as a backend developer at **ETG**, a global flight and hotel booking platform operating in **over 100 countries**, handling very **high traffic at peak** up to **20K** RPS

My main focus is on ongoing process of **migrating critical booking and pricing functionality** from a large Node.js monolith into dedicated backend microservices written in Go,  
driven primarily by scalability and reliability constraints.

I also **led the** backend work on **Hybrid Rate feature**. This one required loads of new business logic, writing more than 100 unit tests as well as designing
new APIs for the frontend.

---

Beyond feature development, a large part of my **day-to-day work** involved

- designing **backend APIs**
- **improving inter-service communication** (including Kafka-based event processing and replacing HTTP with gRPC where lower latency and stricter contracts were required)
- integration testing mostly to validate system invariants (such as no more than one booking can be created at a time under concurrent requests)

I was also involved in observability and production readiness, i did incident management, postmortems

### what are you intrested in ?

Overall, I enjoy solving **complex backend problems** with **many moving parts**.

In recent years, I’ve become especially interested in **distributed systems**, **concurrency**, and **high-load backend services** because this is exactly what a large system with lots
of moving parts is

I’m motivated to keep deepening my knowledge in this area and to share that knowledge through code reviews, technical discussions, and improving system design over time.

### Tell us about your team at your last job. How were responsibilities distributed? How did collaboration work?

- I worked across **several cross-functional teams**, typically \*10–15 people\*\* split into smaller sub-teams depending on the project. All developers were fullstack, so responsibility was shared — if you pick up a feature, you own both the backend and frontend parts.
- Process itself is a modified scrum. What this means is we have **2-week sprints**, **Grooming and planning** at the  
  beginning of each sprint, **no daily standups** - instead we do short project sync meetings twice a week and async
  communication, **team-wide retro** at the end of each sprint
- My role in all of this was: since technical requirements often come from managers words without any docs or technical
  specification, my goal was to ask additional questions, help clarify technical moments, and make sure me and my team
  will be able to hit the goal in the upcoming sprint.

**Please tell us about your main technology stack in recent years.**
Go, TypeScript, Node.js, React

---

### Tell us about your main achievements at work. What are you most proud of?

My biggest achievement was **leading the extraction of the core booking** flow from a large 5-year monolith **into an independent microservices**.

This was challenging because I **wasn’t building something from scratch** — I **had to understand and isolate functionality** that had grown organically over years.

1. **identified boundaries**
2. **write missing tests for existing logic**
3. **extracted and rewrote critical logic**
4. created a **clean architecture**
5. implemented **gRPC communication** full test coverage
6. built **Docker images**, and configured **CI/CD and monitoring** for smooth deployment and manitainability of each
   microservice

The result was a much more stable booking flow and a measurable improvement in performance and conversion. It also enabled the team to ship new booking features much faster.”

---

### Tell us about the most difficult task you had to deal with in the past year. What challenge did you face, and what did you learn from it?

One of the hardest challenges was the **same project — extracting the booking logic** from the monolith.  
As in most **refactor-old-corporate-functionality** tasks, complexity **wasn’t the code** itself, but discovering
**hidden dependencies** that accumulated over years.
The difficulties were:

- dozens of implicit flows,
- undocumented legacy logic,
- tight coupling between modules.

I think what i learned by this is **pacience**, once again, the **importance of testing**, careful deployment with
**blue-green deployment**, **feature-toggles**

---

### If you are applying for a Senior position: do you have experience in building a project from scratch and designing application architecture? Please describe it in more detail.

Yes. At ETG and previously at WowTickets I designed and built multiple microservices from scratch. Even though the logic **partially existed in the monolith**, the architecture
had to be created from zero —

- clean module structure
- APIs
- communication between microservices
- tests
- observability
- Dockerization and smooth deployment, and CI/CD.

In total I've designed **more than 10 services end-to-end**: from business requirements to production deployment.

---

### Please share what motivated you to start looking for a new job and what factors led to your decision to leave your previous role.

I had a **great experience at ETG** — strong team, interesting technical challenges, high-load application. But after **living in the UK for over a year**, I realized
I want to **integrate more** and work in a **fully English-speaking**, fast-paced environment. My team was mostly Russian-speaking and management was English-speaking, so communication wasn’t always aligned with how I want to **grow professionally**.
So now I’m looking for an **English-speaking** team, more **exposure to product discussions**, and a company where I can **contribute long-term.**

---

### What tasks are you interested in working on right now? And what would you like to work on in the future?

I'm motivated by **technical challenges**, X (_high-load_ or _fast-paced_ based on corporate / startup) where good engineering really matters
I enjoy coding and solving complex tasks. Tech stack and the exact is **slightly less** important.
To answer your question straight, im ideally looking for

- Smart engineers in the team
- intresting and tough problems
- room to grow technically and learn

---

### Please clarify if you have any time constraints (vacation, existing offers, or anything else).

No time constraints, notice period is **up to a week, maybe less**

---

### How actively are you searching for a new position ? You have any offers / final stages at the moment ? If so, what is stopping you from taking the offer ?

Yeah, currently I’m currently both having conversations **with several companies**, and I received **some early offers**. But **I set a specific time window** to speak with the companies
I’m genuinely interested in — to compare teams, culture, and technical challenges.

I’m not rushing. I want to make a **thoughtful long-term decision.**

---

### Are you currently working full-time ? What made you consider a contract role ? (thats unusual switching from full-time to contract)

I’m open to both full-time and contract formats since i worked both and both suit me just fine. For me the agreement type is secondary - what matters
is the work, the team, and the opportunity to contribute. I adapt quickly and I’m **comfortable working in a fast-paced environment**, so a **contract role works perfectly fine**.

---

### Leadership, mentoring, hiring developers

I **wasn’t a formal manager**, but I often acted as a **technical lead on projects**.

For example:

- I **led the extraction of the booking engine** from the monolith — defining the architecture, breaking down tasks, coordinating backend/frontend efforts, and guiding other developers through the new service boundaries.
- I mentored several **junior and mid-level engineers**: code reviews, pair programming, explaining architecture decisions, and helping them onboard into complex parts of the system.

- I **coordinated cross-team** work with analytics, DevOps, QA, and designers to ensure smooth delivery.

- I mainly want to write code but i enjoy leadership **that is still hands-on** work

---

## РФ компании

(Дополтительно к тому что выше)

- Вопрос на знание английского: скажи 2 предложения о чем-либо
- Сможешь ли предоставить подтверждение опыта работы ?
- Насколько активно ищещь работу сейчас ? Есть ли оферы на руках ? Финальные стадии ?

---

Now to **Engineering Manager** like questions

---

### Tell me about a time you had to deliver a complex and ambiguous feature. What was unclear about it, how did you approach getting clarity. Who do you talk to in a company. How do you know the moment when everything was clear for the task ?

**What they test with this questions:**

- how to structure uncertainty
- who do you talk to
- how do you turn ideas and discussion into executable work?

**Answer:**

The most recent example is **Hybrid Rates** service. I was responsible for extracting legacy logic into a dedicated
microservice. There were lots of business logic scenarios and we had more than 200 hotels depending on it. So obviously
we could not afford **data loss** and **long downgimes**.

For eash mid-large feature we start by writing **RFC** (Request for comments) file in a separate slack channel with a
deadline. This takes 2-3 days of focused work, pulling differnet people in and the rest of the week of async feedback.  
That **upfront alignment** saved us multiple weeks later by preventing **rework**, clarifying **ownership**,  
**acceptence** criteria, **deployment** strategies.

I communicated to frontend devs, Product manager, other developers to discuss edge cases.

---

### Let's say you've gone through the process of thinking things through, you have a basic understanding and architecture in your head. How do you get a feedback on those, how do you go about getting a thumbs up that this is the right approach ? Do you discuss with other engineers, do you have a process of confirmation ?

**What they test with this questions:**

- consensus vs velocity
- how do you sanity check
- who decides on a “yes”
- your unique role in the process. I listen broadly but im responsible for the final call.

**Answer:**

1. First of all, **RFC** files are time-scoped as i already mentioned.
2. Second, i need to personally make sure that for me and my team **task** is clear, **tradeoffs** are explicitly accepted, there are **no unresolved concerns**, nothing is left for **figuring out later**, risks are understood.
3. Third, we not only discussed the task itself, but the **deployment** strategy as well as testing and metrics we'll
   keep an eye on to make sure everythign runs smoothly.

---

### Do you have a concept of "Architectural decision records" (ADR), concept of staff and principle engineers ?

todo
