

# Problem statement
## Problem domain: Productivity, scheduling, and planning
I have the habit of planning out my routine ahead of time and tracking progress daily to enhance productivity. I spend a lot of time experimenting with productivity tools like Notion, calendar apps, and Trello, trying to keep my life scheduled and organized. Although many of these apps are powerful, I often find them overly complex and not tailored to my personal routine. I often spend over an hour each day updating schedules and tracking progress, making me feel inefficient about my daily workflow.

## Problem: Over-ambitious scheduling that does not match reality
Users of productivity tools (like myself) often create highly detailed schedules filled with ambitious goals. However, in practice, unexpected events, procrastination, or shifting priorities disrupt these schedules. By the end of the day, a planned calendar may look completely different from what actually happened. This mismatch can cause frustration or guilt, even when meaningful work is completed, simply because reality does not align with the original plan. Existing productivity tools cannot quickly and easily adjust to real-time changes in schedules, leaving users with rigid plans that do not reflect the realities of their lives. Most apps emphasize “sticking to the plan” instead of helping users adapt to changes gracefully. I wish scheduling tools were more flexible and forgiving, offering support that adapts to what actually happens and helping me learn how to optimize future plans.


## Stakeholders
1. Productivist (direct users): Mostly students and professionals who hope to boost daily productivity through planning and scheduling. They are the primary users, often overloading schedules and feeling frustrated when plans fail.

2. Peers, teammates, or coworkers (direct users or non-users): People who are involved in the Productivist’s plans or schedules. They are impacted by the problem because when the Productivists’ planning causes missed deadlines or rescheduling, others in their circle are also impacted, causing them to also reschedule to accommodate new changes.

3. Productivity tool developers (non-users): ​​These are companies that create productivity and scheduling apps such as Notion or Google Calendar. They are impacted by this problem because recognizing and addressing this problem could give them valuable insights into designing more adaptive, user-centered tools. However, a highly effective solution (that are not implemented by themselves) may also draw users away from their existing platforms, potentially reducing engagement with their current products.

## Evidence and comparables
1. **Many demand for a better productivity solution:** Many discussions on Reddit are complaining that overly relying on too many productivity tools can hinder work efficiency, and that simple tools with more flexibility could work better. People demanded “[something that doesn’t make me spend more time managing the app than actually doing the tasks](https://www.reddit.com/r/ProductivityApps/comments/1fpcdav/ive_tried_so_many_productivity_apps_over_the_years/),” “[spending more time 'doing' rather than 'planning'](https://www.reddit.com/r/productivity/comments/1bpx3uq/using_too_many_appstools_is_killing_my/),” and “[reclaiming control over my workflow without the unnecessary noise](https://www.reddit.com/r/productivity/comments/1anvcvu/productivity_apps_can_be_toxic/)."

2. [**Planning fallacy is pervasive:**](https://en.wikipedia.org/wiki/Planning_fallacy) Many people systematically underestimate task time compared to reality, which is core to why plans are often too optimistic and scheduled routines go off-script.

3. [**Meetings hijack prime focus time:**](https://www.microsoft.com/en-us/worklab/work-trend-index/breaking-down-infinite-workday) Meetings stretch the day and fragment plans. Microsoft’s 2025 Work Trend Index shows nearly 30% of meetings now span multiple time zones (increased by 35% since 2021), and late-night meetings are up 16% year over year, increasing schedule volatility and off-hours work. The interruptions and fragmentation of daily routines caused by meetings deteriorate workers’ productivity, making them unable to finish their schedules.

![meeting](as1-images/meeting1.png)

4. [**Ad-hoc and last-minute changes are common and disastrous:**](https://www.axios.com/2025/06/17/microsoft-remote-work-meetings) 57% of meetings are ad hoc with no calendar schedule or invite, and 10% are added at the last minute, directly undermining rigid daily schedules.
    > "For many, the workday now feels like navigating chaos — reacting to others' priorities and losing focus on what matters most.” 

5. [**Context-switching is costly:**](https://www.atlassian.com/blog/loom/cost-of-context-switching) It induces stress, distracts attention, and has a high cost of time. Research shows that around 40% productivity is lost from context switching and $450 billion annual loss in productivity globally. An unreasonable schedule may have frequent context-switching, leading to inefficiencies in completing the scheduled tasks.

6. [**Reclaim.ai Solution:**](https://reclaim.ai/) This is a software that automatically performs time blocking to achieve better focus and more productive deep work. While the platform provides powerful features, it remains calendar- and schedule-driven. Users must invest effort upfront in configuration, scheduling, and priority tuning, which can make the setup process complex. Moreover, the app lacks adaptive feedback mechanisms to learn from discrepancies between planned tasks and what actually gets accomplished, limiting its ability to optimize future routines.

7. [**RescueTime Solution:**](https://www.rescuetime.com/) This is a tool that monitors focus sessions and daily activities, offering valuable logging of how time is actually spent and enabling users to reflect on their workflow. While it helps raise awareness of tasks achieved during a day and can improve focus quality over time, it does not function as an adaptive scheduler. Users must still rely on additional platforms to re-adjust and optimize their plans, which introduces extra complexity.

# Application pitch: NEO
> Introducing NEO $\to$ the ONE calendar app you need
## Motivation
Most productivity tools force people into rigid plans that crumble the moment real life interferes, while students and professionals need not just another static planner, but a dynamic companion that learns, adjusts, and help them grow.

## Key features
Neo tackles the scheduling and productivity problems by making daily planning flexible and adaptive:
1. **Adaptive routine tracker**: Productivists simply press Start and Finish on tasks. Neo logs actual time spent on the calendar and automatically reorganizes the day when tasks run long or go expected, so priorities and deadlines still get met without punishing Productivists for delays. This features helps Productivists focus on what matters most and reduces stress when the routine shifts.

2. **Schedule plans**: Productivists create tasks for future days by specifying key fields like task type, priority, deadlines, estimated duration, and flexibility (i.e., split and move booleans). Neo then translate this into a responsive plan that adjusts smoothly as real life unfolds, respecting constraints while mitigating frustrations caused by last-minute changes. 

3. **Smart scheduling suggestions**: drawing from previous schedules and logs on actual activities, Neo identifies time blocks or task types where Productivists are prone to procrastination and flags instances where estimated durations may be unrealistic. Neo then provides evidence-based suggestions for realistic task durations, better focus windows, and time blocks for relax or debrief, empowering users to design plans that work more efficiently in practice.

# Concept design
## TaskCatalog
TaskCatalog manages all user-defined tasks as structured objects with attributes such as duration, priority, dependencies, slack, and deadlines.  In concepts maintains the modularity of Neo's concept design because other concepts, such as TimeBlockPlan, RoutineLog, or AdaptiveSchedule, can reference the task information without duplicating or controlling it directly. In the broader context of Neo, TaskCatalog ensures that constraints (such as dependencies or deadlines) are respected before a task is assigned to time blocks (either by the user or by the adaptive scheduling algorithm).

The generic type parameters are instantiated as follows:
- User is bound to the system’s registered end users, each of whom owns and manages their own set of tasks.
- TaskType represents categories of tasks defined for classification (e.g., Work, Health, Study) and is instantiated in the app where users customize a range of categories.
- Slack is a buffer type for the tasks, instantiated to model permissible deviation margins (e.g., 15 minutes, 1 hour), which influences how adaptive scheduling adjusts plans.

```
concept TaskCatalog [User, TaskType, Slack]

purpose
    allows users to create tasks with different attributes that will get scheduled;
    the attributes will be taken as input by the adaptive calendar algorithm to generate adaptive schedule;

state
    a set of Tasks with
        an owner User
        a taskID String
        a taskName String
        a category TaskType
        a duration Duration
        a priority Number
        a splittable Flag
        a timeBlockSet containing a set of Strings (optional) // these strings are timeBlockIds
        a deadline TimeStamp (optional)
        a slack Slack (optional) // buffer margin for acceptable deviation
        a preDependence containing a set of Tasks (optional) // tasks that it depends on
        a postDependence containing a set of Tasks (optional) // tasks that depend on it
        a note String (optional)

actions
    getUserTasks (owner: User): (taskTable: a set of Tasks)
        requires: exist at least one task with this owner
        effect: returns ALL tasks under this owner
    
    createTask (
        owner: User, taskName: String, category: TaskType, duration: Duration,
        priority: Number, splittable: Flag, deadline?: TimeStamp, slack?: Slack, preDependence?: set of Tasks, note?: String
    ): (task: Task)
        effect
            generate a new taskId that has not been used;
            create a new task owned by owner with the attributes (taskId, taskName, category, duration, priority, splittable, deadline?, slack?, preDependence?, note?), the optional attributes are not required;
            set this task's timeBlockSet as None;
            add this newly created task to postDependence of all tasks in its given preDependence;
            return this newly created task;
    
    assignSchedule (owner: User, taskID: String, timeBlockId: string)
        requires:
            exists task with matching owner and taskId
        effect:
            add timeBlockId to the set of timeBlockSet
    
    verifyTime (taskId: String, intendedStart: Time, intendedEnd: Time): (verified: Boolean)
        requires:
            exists task with taskId
        effect:
            returns True if (the latest end time of any preDependence last is before intendedStart) AND (the earliest start time of any postDependence is after intendedEnd);
            otherwise, return False;
    
    updateTask (owner: User, taskId: String, taskName: String)
    updateTask (owner: User, taskId: String, category: TaskType)
    updateTask (owner: User, taskId: String, duration: Duration)
    updateTask (owner: User, taskId: String, priority: Number)
    updateTask (owner: User, taskId: String, splittable: Flag)
    updateTask (owner: User, taskId: String, deadline: TimeStamp)
    updateTask (owner: User, taskId: String, slack: Slack)
    updateTask (owner: User, taskId: String, postDependence: Number)
    updateTask (owner: User, taskId: String, preDependence: Number)
    updateTask (owner: User, taskId: String, note: String)
        requires:
            exist a task with this taskId and the owner matches the given owner
        effect:
            update the given attribute of this task;
            if preDependence is changed, also modify the postDependence of related tasks;
            if postDependence is change, also modify the preDependence of related tasks;
    
    deleteTask (owner: User, taskId: string)
        requires:
            exist a task $t$ with this taskId;
            task $t$ has no postDependence;
            task $t$ has a matching owner;
        effect:
            remove task $t$ from Tasks
```

## ScheduleTime
The ScheduleTime concept manages the user's intended allocation of tasks onto their calendar. I maintained the modularity of Neo's concept design by specifying that TaskCatalog defines the content and attributes of tasks (what needs to be done, with what priority, duration, or constraints), and ScheduleTime manages the tasks' placement in the day (i.e., when they are intended to happen). Tasks remain abstract objects until concretely scheduled into time blocks, and time blocks remain flexible containers that can be reorganized without altering the underlying task definitions.

In the broader app architecture, ScheduleTime plays a crucial role in showing users their intended schedule. The Today UI views (from the wireframes) draw directly from ScheduleTime to visualize planned routines. ScheduleTime also enables adaptive scheduling. When deviations occur in the actual routine, the adaptive scheduler updates or reassigns tasks within these time blocks. By modularizing plan scheduling separately from task attributes and adaptive scheduling, the design supports scalability and reduces complexity.

We have a generic parameter User, which is bound to the system’s registered end users, each of whom owns and manages their own set of tasks. We also have a generic parameter TaskId, which are references stored in each time block. TaskId links directly to TaskCatalog, but without duplicating task details.

```
concept ScheduleTime [User, TaskId]

purpose
    manages users' intended schedule of future tasks by allowing them to assign tasks to each time block

principle
    users can allocate tasks to one or more time blocks
    the time blocks reflect the user's intended schedule of the day

state
    a set of TimeBlocks with
    a timeBlockId String // this is a unique id
    an owner User
    a start Time
    an end Time
    a taskIdSet set of TaskId

actions
    getUserSchedule (owner: User): (schedule: a set of TimeBlocks)
        requires:
            exists at least one time block under this owner
        effect:
            return a set of all time blocks under this owner ending before day's end
        
    addTimeBlock (owner: User, start: Time, end: Time)
        requires:
            no time block exists with this owner, start, and end
        effect:
            create a new time block $b$ with this owner, start, and end, and empty taskIdSet
    
    assignTimeBlock (owner: User, taskId: TaskId, start: Time, end: Time): (timeBlockId: String)
        requires:
            if exists time block b with matching (owner, start, end), then timeId is not in b.taskIdSet
        effect:
            if b doesn't exist, create it with owner, start, and end (same as addTimeBlock)
            add taskId to b.taskIdSet;
            return b.timeBlockId;
    
    removeTask (owner: User, taskId: TaskId, timeBlockId: String)
        requires:
            exists a time block with matching owner and timeBlockId;
            taskId exists in this time block's taskIdSet;
        effect:
            remove taskId from that block's taskIdSet

```

## RoutineLog
The RoutineLog concept captures the reality of what a user actually does during the day, complementing the planned schedules stored in TaskCatalog and ScheduleTime. Each session records start and end timestamps, optional links to planned tasks, and metadata such as pause flags or interruption reasons. This concept ensures that the system has an accurate log of the user’s real activity, which is critical both for the user to reflect on the daily routine (via Compare views) and for feeding the adaptive scheduling algorithm.

In the overall app, RoutineLog logs the actual routine against which intended schedules can be compared. AdaptiveSchedule uses this data to adjust future allocations when tasks run long or get interrupted. Insight features also depend on RoutineLog to identify patterns, such as frequent pauses or underestimated durations. By modularizing the capture of actual sessions, RoutineLog cleanly separates the recording of reality from both planning (ScheduleTime/TaskCatalog) and adaptation (AdaptiveSchedule), ensuring independence and clarity across the system’s design.

The generic parameters are instantiated as follows:
- User is bound to the system’s registered end users, each of whom owns and manages their own set of tasks
- Task refers to task objects managed by TaskCatalog; when linked, a session provides evidence of whether the task was completed as intended. When no task is linked, RoutineLog will support ad-hoc or unscheduled work.

```
concept RoutineLog [User, Task]

purpose
    capture what actually happened throughout the day as time-stamped sessions, optionally linked to plans;
    enable adaptive scheduling and allow users to reflect on their plans;

principle
    after a user starts and finishes a session, the system records its actual start and end, and can associate it with a planned task

state
    a set of Sessions with
        an owner User
        a sessionName String
        a sessionId String    \\ this is an unique ID
        an isPaused Flag
        a start Time (optional)
        an end Time (optional)
        a linkedTask Task (optional)
        an interruptReason String (optional)
    
actions
    createSession(owner: User, sessionName: String, linkedTask?: Task): (session: Session)
        effect:
            generate a unique sessionId
            create a session owned by owner with sessionName
            if linkedTask is provided, assign it to this session
            assign start and end for this session as None
            assign isPaused as False
            assign interruptReason as None
    
    startSession(owner: User, session: Session)
        requires:
            session exists and is owned by owner
        effect:
            get the current timestamp
            set session.start = current time stamp
    
    endSession(owner: User, session: Session)
        requires:
            session exists and is owned by owner
        effect:
            get the current timestamp
            set session.end = current time stamp
    
    pauseSession(owner: User, session: Session, interruptReason: String)
        requires:
            session exists and is owned by owner
        effect:
            get the current timestamp;
            set session.end = current time stamp;
            set session.isPaused = True;
            set session.interruptReason = interruptReason;
    
```

## AdaptiveSchedule
The AdaptiveSchedule concept is responsible for keeping the user’s plan responsive when reality diverges from intention. Whereas TaskCatalog and ScheduleTime represent the static plan, and RoutineLog records what actually happened, AdaptiveSchedule bridges the two by generating updated schedules whenever tasks run long, are interrupted, or priorities shift. It ensures that deadlines, priorities, dependencies, and slack tolerances are respected while still optimizing for what can realistically be accomplished.

Within the context of the app, AdaptiveSchedule plays the role of a dynamic optimizer: when invoked (i.e., when the user clicking "Optimize Schedule" in the UI), it computes a new allocation of tasks to AdaptiveBlocks. These AdaptiveBlocks serve as a modified plan that can be written back into ScheduleTime. This design maintains modularity: AdaptiveSchedule never edits task definitions, changes planned schedule, or logs sessions, but instead produces an updated schedule that follows the rules defined elsewhere.

The generic parameters are instantiated as follows:
- User is bound to the system’s registered end users, each of whom owns and manages their own set of tasks
- Task refers to task objects stored in TaskCatalog, each with the attributes (priority, splittable, deadline, slack, dependencies) that the adaptive algorithm evaluates
- TimeBlock refers to planned blocks from ScheduleTime that provides information on the planned schedule


```
concept AdaptiveSchedule [User, Task, TimeBlock]

purpose
    keeps the schedule responsive by moving, canceling, or creating tasks at future time blocks when reality diverges to ensure that highest priority tasks are achieved first, optimizing productivity

principle
    when actual sessions overruns or diverges from the plan, the scheduler adjusts subsequent blocks of planned tasks while referencing the tasks' properties like splittable and slack, and respecting priorities, deadlines, and dependence

state
    a set of AdaptiveBlocks with
        a timeBlockId String // this is a unique id
        an owner User
        a start Time
        an end Time
        a taskIdSet containing a set of taskIds
    

actions
    addTimeBlock (owner: User, start: Time, end: Time)
        requires: no adaptive time block exists with this owner, start, and end
        effect:
            create a new adaptive time block $b$ with this owner, start, and end;
            assign $b$ an empty set of tasks;
    
    createAdaptiveSchedule (owner: User, taskTable: a set of Tasks, plannedTimeBlocks: a set of TimeBlocks)
        requires:
            all time blocks in plannedTimeBlocks has a start that is after the current time
        effect:
            referencing information from tasks' attributes (priority, splittable, deadline, slack, preDependence, postDependence) and the current schedule in plannedTimeBlocks, adaptively generate a new schedule by assigning active tasks to the corresponding AdaptiveBlock under this owner
```

## Synchronizations
1. Add new task
2. Optimize schedule
3. Start a session with a planned block
4. Start a session with a new task
5. Pause/interruption of a session
6. complete a session

### Add and remove task
```
sync createTask

when
    Request.addTask(user, taskName, category, duration, priority, splittable, deadline?, slack?, preDependence?, note?)

then
    TaskCatalog.createTask (owner: user, taskName, category, duration, priority, splittable, deadline?, slack?, preDependence?, note?)
```

```
sync scheduleTask

when
    Request.addTask(start, end)
    TaskCatalog.createTask (): (task: Task)
    TaskCatalog.verifyTime (taskId: task.taskID, intendedStart: start, intendedEnd: end) : (verified)
        requires: proceed only if verified is True

then
    ScheduleTime.assignTimeBlock (owner: user, taskId: task.taskId, start, end): (timeBlockId)
    TaskCatalog.assignSchedule (owner: user, taskId: task.taskId, timeBlockId)
```

### Optimize schedule
```
sync optimizeSchedule

when
    Request.optimizeSchedule(user)
    TaskCatalog.getUserTasks(owner: user): (taskTable)
    ScheduleTime.getUserSchedule(owner: user): (schedule)

then
    AdaptiveSchedule.createAdaptiveSchedule (owner: User, taskTable, plannedTimeBlocks: schedule)
```

### Start a session with a planned task
```
sync startSessionWithPlannedTask

when
    Request.startPlannedSession(user, sessionName, linkedTask)
    RoutineLog.createSession (owner: user, sessionName, linkedTask): (session)

then
    RoutineLog.startSession (owner: user, session)
```


### Start a session with a new task
```
sync startSessionWithNewTask

when
    Request.startNewSession(user, sessionName)
    RoutineLog.createSession (owner: user, sessionName): (session)

then
    RoutineLog.startSession (owner: user, session)
```

### Pause session
```
sync pauseSession

when
    Request.pauseSession(session, interruptReason)

then
    RoutineLog.pauseSession (owner: user, session, interruptReason)
```

### Complete session
```
sync completeSession

when
    Request.completeSession(user, session)

then
    RoutineLog.endSession(owner: user, session)

```

# UI Sketches
There are three main pages/views in Neo: Today, Compare, and Record. I will go through the UI sketch for each, and what purposes each view serves below. Each of the UI design is annotated with pointers, and more detailed explanation are provided below the UI.
## View 1: Today
### Show today's schedule
This page displays a clean timeline of the user's schedule for the current day, showing tasks' schedule and progress. The key component is a timeline with tasks stacked along time slots. Each task shows priority and progress bar.

This simply visualization of the daily schedule shows clear indicators of progress and priority, reducing decision fatigue.

![today](as2-ui/today_ui.png)

### Add task
The add task panel allows users to create planned tasks by entering essential information (task name, category, duration, priority, and optional settings like deadline, buffer/slack, prerequisites, and postrequisites). These information will help the algorithm to produce more optimized adaptive schedule.

![today](as2-ui/addtask_ui.png)


## View 2: Compare
This page is key to Neo. It shows a side-by-side comparison of the planned schedule versus the actual routine/sessions the user recorded. It highlights perfect matches, mismatches, and unplanned tasks, and shows how the actual routine deviates from the plans.

Instead of punishing users for failing to "stick to the plan" and leave them with no solution for the resulting chaos, Neo reframes deviations as learning opportunities. This page gives visibility into how real life differed from plans. Neo also offers a solution to optimize the schedule. By clicking on the "Optimize Schedule" button, Neo re-adjusts remaining tasks/plans in the day to account for deviations, helping users meeting top priorities and deadlines. Neo helps users update the day dynamically.

![compare](as2-ui/compare_ui.png)

## View 3: Record
### Start and end session
In the Record page, the user logs current work session in real time. The page will track how long the user spends on a chosen task, and the actual session block will be reflected in the Compare page.

The goal of this page is to make it easy to record real-time sessions, reducing the friction in logging actual routines, and provide data for adaptive scheduling.

![record](as2-ui/record_ui.png)

### Pause session
Interruptions are a huge source of productivity loss. If a session is interrupted (i.e., if the user clicks "PAUSE"), the app learns *why* users go off-track and uses that insight to refine future scheduling suggestions (such as factoring in short breaks, buffer times, or ad hoc meetings). 

![record_palse_ui](as2-ui/record_palse_ui.png)

# User Journey
Jay is a Productivist. As a product manager at Microsoft, Jay balances cross-team meetings, roadmap planning, and individual focus work every day. To stay on top of everything, Jay creates detailed schedules of his daily plan in Notion Calendar, and asks his colleagues and clients to book meetings through this platform too.

However, this Tuesday mid-afternoon is not the first time Jay realizes his ambitious and organized schedule of the day collapsed -- after three ad-hoc meetings (two of which run long), two urgent pings from his colleagues, and one new task from his lead. His focus work gets completely postponed, and he is uncertain whether he can still finish his tasks on time. At that moment, Jay decides to try Neo, a scheduling tool that flexes with reality, helping him navigate the overwhelming situation of planned schedules not matching the reality, making schedules more adaptive and effective.

Using the [Add Task panel](#add-task), Jay quickly enters this remaining tasks for the day, including "Write Product Spec," "Team Sync," and "Review Customer Feedback." He assigned each task a priority, duration, and category, and allocates these task blocks onto the timeline. For “Write Product Spec,” Jay sets a hard deadline and allows it to be splittable across multiple sessions.

On the [Today](#show-todays-schedule) page, Jay sees a simple and intuitive timeline that shows what he is expected to work on for the remainder of the day. The page shows him meetings in fixed slots, focus tasks filling open time, and color-coded priorities.

When Jay begins a session on "Writing Product Spec," he opens the Record Session, chooses the "Writing Product Spec" task, and clicks on the start session button. Halfway through finishing his work, he gets interrupted by an urgent call from the engineering team. Jay pauses the session and logs the cause using the [Pause Session interface](#pause-session), selecting "Interruption — Call/Meeting." This not only pauses the timer but feeds Neo data about interruptions, helping future planning.

By afternoon, Jay looks at the [compare interface](#view-2-compare) and notices that too many interruptions have piled up, and it is clear that not all of his planned tasks can be completed. Instead of scrambling or feeling overwhelmed, Jay clicks the Optimize Schedule button on the [compare page](#view-2-compare). Neo adaptively and automatically reschedules the remaining timeline, pushing non-critical tasks into tomorrow, while prioritizing hard deadlines and high-priority work for the rest of the day. Neo also learns from Jay's logged sessions through out the day, analyses his focus pattern and energy cycle, and suggests short breaks to boost Jay's overall productivity. This gives Jay confidence that the most important outcomes will still be achieved.

At the end of the day, Jay revisits the [compare page](#view-2-compare). The timeline shows what matched perfectly in green, what deviated in red, and what got dynamically rescheduled in grey. Instead of feeling guilty about unfinished items, Jay sees a rationalized schedule where the most important tasks were accomplished despite interruptions.

Neo reframes deviations as learning opportunities, and offers Jay insights like: “Spec writing tasks take 1.5x longer than estimated.” For Jay, this means ending the day with clarity rather than guilt. He learns about his own focus pattern, energy cycles, and work habits. Most importantly, Neo helps him finished most of his high-priority tasks. Over time, Jay builds more realistic schedules and feels more effective as a busy product manager.