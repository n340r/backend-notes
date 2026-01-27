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

### ❓ Tell me about a time you had to deliver a complex and ambiguous feature. What was unclear about it, how did you approach getting clarity. Who do you talk to in a company. How do you know the moment when everything was clear for the task ?

**What they test with this questions:**

- how to structure uncertainty
- who do you talk to
- how do you turn ideas and discussion into executable work?

**Answer:**

The most recent example is **Hybrid Rates** service. I was responsible for extracting legacy logic into a dedicated
microservice. There were **lots of scenarios** and we had **more than 200 hotels** depending on it. So obviously
we could not afford **data loss** and **long downgimes**.

For eash mid-large feature we start by writing **RFC** (Request for comments) file in a separate slack channel with a
deadline. This takes 2-3 days of focused work, pulling differnet people in and the rest of the week of async feedback.  
That **upfront alignment** saved us multiple weeks later by preventing **rework**, clarifying **ownership**,  
**acceptence** criteria, **deployment** strategies.

I communicated to frontend devs, Product manager, other developers to discuss edge cases.

---

### ❓ Let's say you've gone through the process of thinking things through, you have a basic understanding and architecture in your head. How do you get a feedback on those, how do you go about getting a thumbs up that this is the right approach ? Do you discuss with other engineers, do you have a process of confirmation ?

**What they test with this questions:**

- consensus vs velocity
- how do you sanity check
- who decides on a “yes”
- your unique role in the process. I listen broadly but im responsible for the final call.

1. First of all, **RFC** files are time-scoped as i already mentioned.
2. Second, i need to personally make sure that for me and my team **task** is clear, **tradeoffs** are explicitly accepted, there are **no unresolved concerns**, nothing is left for **figuring out later**, risks are understood.
3. Third, we not only discussed the task itself, but the **deployment** strategy as well as testing and metrics we'll
   keep an eye on to make sure everythign runs smoothly.

---

### ❓ Do you have a concept of "Architectural decision records" (ADR), concept of staff and principle engineers ?

**What they are asking:**

- Do you think about how architectural decisions scale beyond one **feature / year / team**
- How decisions are **recorded / revisited / challenged**
- If you have a team where decisions **survive people leaving** the company”

Staff, Principal engineers - more about carrying experience across teams

**ADR** - Architectural decision records

1. Captures the final decision
2. Records why alternatives were rejected
3. Stable reference for future teams

Difference between request for comments file

- **RFC** - decision in progress
- **ADR** - decision that must outlive the feature and people

Staff = “Teams don’t step on each other.”
Principal = “We won’t regret this in 3 years.”

Yes, we do rely on **ADR** not to repeat other people's mistakes and have once in 2 week Staff engineer meeting to listen to ideas

---

### ❓ Can you tell me about a time you were responsible for leading a complex feature. Who were those people you had underneath/with you that you delegated to. What challenges you faced and how do you overcome them. Have you ever worked on a larger team, did you have the opportunity where someone has gone "this is your big project and people to help you out, you are responsible and accountable for delivery, communication, planning and everything"

**What they are asking:**

- Basically how much of a responsibility you have
- how you structure work

The most unclear and ambiguous task i had to deliver recently was the backend for **Hybrid Rate** feature

I did not have a formal authority of being called lead, but I was **responsible for delivery**.

At the time we had just hired a less experienced **mid-level engineer**.

I broke the work down so that

**I owned** the high-risk crucial parts:

- architecture
- data contracts
- deployment strategy
- database migrations

**I delegated:**

- endpoints
- tests
- new business logic needed.

The way we progressed quite fast without loosing quality and going into miss-communication:

- I **always discussed** in advance on a high level the \**approach he would take*8 to implement something rather than rushing right in and giving me a surprise on a code review
- Always be available for the **questions**.
- Frequent **code reviews**, splitting code into tiny pieces.

The way we kept business up to date on our work is basically Milestones on our work

- migrate this piece of data
- migrate those endpoints
  make sure gRPC is set up and running

---

### ❓ Can you tell about a time you improved observability? What was missing and what impact did your changes have.

**What they are testing:**

- Are you just a **passive consumer** of observability or do you actually care ?
- Do you understand the significance of **observability** ?

**Obviously** you should know: metrics, logs, traces + triggers/alerts

Answering **framework:** “We didn’t have **signal X**, which caused **blind spot Y**, so we **added metric/alert Z**, which reduced risk”

I'll give you three examples of answering this question:

---

1. 💥 For each Kafka event we loaded **all bookings of a hotel** into memory.
   During **Black Friday** some hotels had **huge** booking history, so **memory grew** unbounded and the service was **killed by OOM**.

We had an OOM incident where a pricing service loaded all bookings of a hotel into memory for Kafka events. During Black Friday this caused unbounded memory growth

We obviously changed this ineffician algorithm, started processing those bookings in batches.

**What i added:**

- Kafka consumer lag
- Memory usage of consumers

---

2. 💥 Missing **204 statuses**

Backend changed an availability endpoint `GET /availability?hotelId=123&dates=...` toreturn `204` for **no-data** but we did not have
a metric for it. Frontend **missed the task** on upating their status to `204` - maybe task got lost due to it being
**seemingly insignificant**.

This caused \*8request amplification** and OOM in the **availability\*\* service

After backend with a new return status rolled out

**What i added:**

- Status code ratio metrics (`200`/ `204`/ `4xx`)

---

3. 💥 Alert fatigue during peak load.

**Alerts** and triggers weren’t adjusted” along with scaling the system.  
Engineers **ignored important** alerts because they **drowned** in **alert noise**.

**What i added:**

- Scaling-aware alerts. Change triggers with system scaling
- **percentage-based** error rates rather than static numbers

---

### Have you ever used observability to make development decisions? What metrics were you looking at, how did you understand them and how did those change your understanding, your architecture. Was there ever a moment where you looked at the metric and went "oh, this requires me to make a change in the system"

**What they are testing:**

- Keeping an eye on **costs** and **efficiency**
- **Proactive behaviour** (not just incidents and firefighting)
- **Mature** point of view: don't rewrite without a clear reason.

---

1. 💥 Option 1. Memory costs ramping on staging.

I noticed an **unusual growth** in **S3 cloud storage** and **snapshot costs** for staging, which was disproportionate to actual usage.  
Turned out we had **useless** stage **Kafka topic backups** and **database snapshots** on staging because they provided no value.

---

2. 💥 Option 2. SQS + Kinesis

Since we process a lot of use behavior from search and hotel managment, we had a legacy `SQS` queue + `Kinesis` streaming pair for those.

I actually saw the usage cost for both of those, **validated the change** with a staff engineer and decided to migrate event processing logic with Kafka
becaues **throughput allows** it and we win pricing-wise.

What i did step by step:

- I created a **new topic** in an **existing** kafka cluster
- **moved consumers**
- validated reads
- moved **producers**,
- set up a basic set of metrics for new consumers
- turned off SQS, Kinesis

### How do you prepare your code so that it is comparable with metrics required ? How do you ensure that there is technical implementation to achieve metrics, observability ?

**What they are testing:**

- Is observability an afterthought or is it an integral part of development process ?

We obviously **do not write** business logic for metrics but observability is **never an afterthought**.

To me **this means** developing the code so that there are **explicit boundaries** and **separation of concerns**.

Monitoring one thing or another is **only possible** when there are **boundaries**. When everything happens in one function or one service or one API request then it would
be more difficult for us to find out what exactly is wrong

We often have issues with performance and metrics help us identify wether that is **kafka memory**, **backpressure** / **consumer lag**, **database** is a bottleneck

### Switching away from the tech a little bit. Tell me about a time you've really understood your end-users. How has your knowledge or understanding of end-users changed the way you designed or implemented a feature ?

Please help me correctly structure this, choose the best option

1. **👥 Option 1: Promo banner placement**.

We were testing a **promo-banner** on the search-page’s frontend and wanted to make it’s **conversion** as high as possible without making it too **intrusive**.  
Users **skip intrusive irrelevant** commercials and promos.

What we did is along with **design and analytics team** we started to tweak the API so that banner placement and content is determined on certain user behavior
and info that we received from the frontend.

> 💡 "What metrics did you use to determine banner placement ?" - New users: no banner, Returning users - small banner, Ones who searched multiple times - promo shown once

2. **👥 Option 2: Conversion rate drop**.

I saw a noticeable drop in search to book **conversion** and found out that at the same time **search response times **increased** due to a **database issue\*\*.

That basically **reinforced the idea** that search is not a feature, it is a **core user expectation**. As a result of that we not only resolved the existing issue,  
we improved **caching**, **optimised** DB, added **tighter latency metrics** for search and filtering related api-endpoints.

> 💡 "How did you measure conversion ?" - We count how many users with same `tenent_id` completed the funnel `search → view → book → pay`. This is not too complicated just requires consistent event tracking from **multiple services**.

> 💡 "What was the issue with DB ?" - The database became a bottleneck during peak load and needed scaling.

> 💡 "You said you added caching, what kind of caching ?" - We added **availability** data caching with short TTL. Search result caching existed already.

### How do you normally get this info about users ? Is it through PM ? Is the PM doing the investigation for you? Say you have a new project on the go and they're building "assistance portal" and when it comes to calls we need to understand when the spike of usage comes in, what type of user personas are going to log in, what devices they might be using, retention rate, features of the system used more than others, etc, who is doing all of that research before you actually proceed and development starts ? Is this "discovery phase" done only by the PM or by developers as well ?

This **depends** largely on a project we're building but **most of the time**, Discovery process is actually shared between **PM, Analytics**.
Those people don’t get the data out **of nowhere because** we as engineers provide this data to them via request metadata.

I can tell you how data gathering works.

**How data gathering works:**

- Services emit evens,
- Kafka consumes those analytics need
- Store it into ClickHouse
- This is later **used by analytics** to build their dashboards etc.

(The following is already diving too deep into technical, but just in case).

To give you an exact examples of what we do on the backend:

```json
{
  "event": "SearchCompleted",
  "user_type": "guest",
  "device": "mobile",
  "latency_ms": 320,
  "results_count": 14
}
```

Another thing we do is extract **user-agent** (UA) already present in the header, we extract it's useful parts (device type, OS, or browser) in int the api-gateway.
And store those into **attach those** to domain event like `SearchCompleted` and store it into ClickHouse.

### We hold an enormous amount of data. Hold very sensitive PR data: medical records, criminal records of people. What is your philosophy on security and can you give me an example of security on each layer: DB, transport, backend, frontend etc. and how you would go about implementing security on each one of those. GDPR data, do you do anything special to that sort of data (like credit card data)

**What they are asking:**

- They likely have a security team and understand that this is a **separate topic** of it's own.
- They want to test if i understand the basics of **security on each level** as a dev.

I rely on IT security team we have to **set the policy** and rules and I basically **implement** this all in code.
I do **understand** the **most crucial** security concerns on each level.

To give you an example:

- **DB**:
  - guest users can’t do admin stuff
  - encrypted backups
  - encrypted passwords
  - dont store data you dont need.

- **Transport layer**:
  - TLS everywhere
  - strict auth
  - do not return DB-layer errors to en end-user

- **API layer**:
  - rate limiting
  - secrets management
  - Authn/Authz
  - do **not** give users more data then they need so that they dont start reverse engineering our backend table structure.

- **Frontend**:
  - XSS (cross-site-scripting), avloid pasting unsanitised HTML directly
  - use secure cookies
  - CSRF tokens

- **GDPR**:
  - only store what is needed: do not store all user's info if a simple flag **purchased/didn't** is enough for our system
  - be able to tell **who accesed** data and **when**.
  - **automatically delete** after X period of time.

??? A few more examples that I would like you to polish and help me fit into an answer:

We actually have ADR documents that state how developers should work with security. They contain things I mentioned and many more. We always refer to them

??? Do not actually understand what those mean and why they are so crucial for GDPR data: minimization, auditability, and retention/deletion requirements

### Collaboration and mentorship. Have you ever been given an engineer and realised "oh this person does not know some technology from the stack" or "oh he does not have this soft skill or whatever" and you have to spend time mentoring them on something or has it been just "day-to-day communication and task discussions"

We hired a **mid-level** engineer who was new to parts of our stack, so I acted as a **buddy** during onboarding.

I focused on helping him understand our **system**, **business domain** and working style. I set up regular **1x1 meetings** for us to catch up regularly.

One specific gap was **Kafka** that he did not work before. So I had to teach him `offsets`, `error handling`, `retries` etc.

Once I understood that he became more confident I deliberately **stepped back** and we moved to normal code reviews and async communication

### Tell me a failure of a mistake you did on a project, what did you learn, how did you prevent it from happening again

I was responsible for moving that does **all** messaging logic (Login codes, booking confirmations, etc) into a dedicated **Messaging service**.  
Service had to be deployed fast so i set up simple HTTP communication for all of those message requests.

During peak traffic on Fridays, the service would **occasionally go down**. The problem **wasn’t detected by monitoring** — I only found out
after a **user contacted support**, which was a serious gap.

The root causes were twofold:

1. There were **no health or alerting** signals for the service
2. It was **tightly coupled** via synchronous HTTP calls, so traffic spikes caused cascading failures.

**What i did:**

- Redesigned the service to consume events asynchronously via **Kafka**. That **decoupled** it from traffic bursts and made failures visible immediately.
- I added proper **availability** and **throughput alerts**.

The key lesson for me was that production issues aren’t always about **fixing the bug** — they’re about fixing the system so the same class of failure wont’t **happen again**.

### 💥 Biggest mistake EVER. It might not be your fault, just tell me about the biggest explosion you've seen at a project, what the result of it was

A separate team were building **data-pipelines** for analytics that are processed via `Kafka`, `Snowflake`.
They wanted to test **production-like data volumes** on staging.

**They did two mistakes**:

1. Forgot to set up a **spending limit** and **alert** on staging because why bother that is temporary anyway
2. Forgot to shut staging off from production volumes.

`Snowflake` compute and storage costs caused by **production-scale** data continuously flowing through staging, combined with **frequent processing jobs** resultee in
a company receiveing an additoinal `50K $` bill from cloud provicer one day. We later negociated it to lower price but not good anyway.

**Lesson I took from it:** Be careful with any big volumes of data, remember that prod, staging both run on real machines that cost real money, never fire and forget.

### Incident commander. Do you have this process in the team ? Who manages the incident, what is the team that gets pulled in to help ? Is there an “incident coordinator” and other people who “do the actual work” for the incident ?

We have **weekly on-call rotation**.

When you are on shirt this is you **main** responsibility and all new features are secondary.

Once incident finishes you are **responsible** for conducting **post-mortem call**. This is **not** to punish or to blame.  
In a large organization where everything is highly systemised, incidents help us detect where our **architecture** or **assumptions** went wrong.

On-call engineer acts as the incident commander. That means I’m responsible for:

- coordination
- assessing impact
- deciding priorities
- pulling in the **right people**: DevOps, SREs
- Keeping communication **clear** — not necessarily fixing everything myself.
