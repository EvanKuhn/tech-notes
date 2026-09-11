# Interview Framework ⭐⭐⭐

* **Step 1: Understand the problem (3 \- 10 mins)**
    * **Slow down** and think deeply
    * **Ask questions** about **requirements**, assumptions, etc
    * Example questions:
        * What features are we going to build?
        * How many users does the product have?
        * Traffic volume? Expected scale over 3/6/12 months?
        * Web vs mobile?
        * etc...

* **Step 2: Propose a high-level design, get buy-in (10 \- 15 mins)**
    * **Collaborate** with the interviewer. Ask for feedback.
    * **Draw diagrams**, do some back-of-envelope calculations
    * Step through a few **use cases** or flows. Consider edge cases
    * **Communicate** as you work

* **Step 3: Design deep-dive (10 \- 25 mins)**
    * At this point you should have:
        * Agreed on overall goals and scope
        * Sketched out a high-level design
        * Obtained feedback
        * Had some initial ideas to focus on for the deep dive
    * Work with interviewer to identify and prioritize components in the architecture
    * **Time management** is essential
        * Skip unnecessary details

* **Step 4: Wrap up (3 \- 5 mins)**
    * Things you may cover here:
        * Recap of the design
        * Identify bottlenecks
        * Review error cases (eg: server failure, network failure)
        * Operational: metrics, error logs, deployment
        * How to handle the next scale curve (eg: 1M \=\> 10M users)
        * Propose other refinements
    * Dos:
        * Ask for clarification
        * Understand the requirements
        * Communicate\! Speak your thoughts out loud
        * Suggest multiple approaches
        * Bounce ideas around, get buy-in
        * Don't give up\! 🙂
    * Don'ts:
        * Don't be unprepared for typical questions
        * Don't jump into a solution without clarifying requirements and assumptions
        * Don't go into too much detail on a single thing
        * Don't hesitate to ask for hints
        * Don't remain silent

**From ChatGTP:**

1. Phase 1: requirements
    1. Clarify requirements
    2. Estimate scale
2. Phase 2: initial solution
    1. Define APIs
    2. Draw the high-level architecture
    3. Explain the data model
3. Phase 3 / 4: deep dive \+ wrap up
    1. Identify bottlenecks
    2. Scale each bottleneck
    3. Discuss reliability
    4. Discuss consistency
    5. Explain tradeoffs
