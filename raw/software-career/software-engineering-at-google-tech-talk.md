# Software Engineering at Google

> Source: User-provided transcript file: r64iT7jhp2s_Google.md
> Collected: 2026-08-16
> Published: Unknown

Hello and welcome to today's ACM Tech Talk.

This webcast is part of ACM's commitment to lifelong learning and professional development,

serving a global membership of computing professionals and students.

I'm Hyrum Wright and I'm excited to moderate today's talk with my friend and colleague Titus Winters.

I'm a senior staff engineer at Google where I lead the code health team.

My team is responsible for the maintainability of Google source code,

ensuring the scalable evolution of billions of lines of code.

In clip In collaboration collaboration with Titus,

I'm one of the editors of the book Software Engineering at Google and

occasionally I get to teach at Carnegie Mellon.

I'm also an ACM member.

For more on my background, check out the bio widget on your screen.

For those of you who may be unfamiliar with ACM or what it has to offer,

here is more information.

ACM offers educational and professional development resources that bolster skills and enhance career opportunities.

Our members can stay competitive in the constantly changing world of

computing with a range of ACM learning center resources at learning.acm.org.

You can see some of the highlights on your screen.

ACM recognizes the role of computing in driving the innovations that

sustain competitive competitiveness in a global environment.

ACM provides access to the ACM digital library,

the world's most comprehensive database of computing literature.

Leading publications and global conferences that draw top experts on a broad spectrum of computing topics.

Support for education research including curriculum development,

teacher training, the ACM Turing and ACM prize computing awards.

And the ACM Code of Ethics,

a collection of principles and

guidelines designed to help computing professionals make ethically responsible decisions in professional practice.

AC ACM enables its members to solve critical problems using new technology that enriches our lives and

advances society in the digital age.

So before we get started,

I'd like to quickly mention a few housekeeping items shown on the slide in front of you.

If you have questions at any time,

please type them into the Q&A box and click submit.

I'll organize the questions as Titus speaks and we'll try to get to as many as we can,

but given the attendance today we can't guarantee we'll be able to answer every question.

This session is being recorded and will be archived.

You'll receive an automatic email when

it becomes available and check learning.acm.org for updates on this and upcoming webcasts.

At the end of the presentation, you'll see a survey open on your screen.

Please take a minute to fill it out to help us improve our tech talks.

You may also open the link to the survey at any time from the resources window.

Today's presentation is Software Engineering at Google by Titus Winters.

Titus is a senior staff engineer at Google where he has worked since 2010 and we've worked together since 2012.

At Google, he is the library lead for Google's C++ code base.

250 million lines of code code that will be edited by 12,000

distinct engineers every month.

He served He served for several years as the chair of the sub committee for the design of the C++ standard library.

For the last 10 years, Titus and his teams have been organizing,

maintaining, and evolving the foundational components of Google's C++ code base using modern automation and tooling.

Along the way, he has started several Google projects that are believed

to be among the top 10 largest refactoring efforts in human history and

he usually ropes me into helping him out with them.

That unique scale and perspective That unique scale and

perspective has informed all of his thinking on the care and feeding of software systems.

His most recent book His most recent project is the book Software Engineer Engineering at Google,

also known as the flamingo book,

published by O'Reilly in early 2020.

Titus is also an ACM member.

So, Titus, go ahead and take it away.

Thank you so much.

And thanks everyone for having me.

Uh I'm very excited to share this presentation with all of you and uh one,

it's a very large audience and that's very exciting and it's great publicity, of course.

Uh but two, it's just a talk that I I really have a deep personal connection to cuz in many respects,

this talk is directly derived from the original pitch uh presentation that Hiram and

I made to a VP years ago when we were starting this project.

So, this is kind of bringing things full circle.

So, anyway, I hope you enjoy it.

So, when we go out to customer visits or give conference talks,

I'm sometimes asked, "What's the secret?

Why is Google good at software?"

And that assumes that we are,

which sometimes is not true,

but bear with me for the sake of argument.

I gave a talk at a conference a few years ago that included the following pitch:

"For the low, low price of $5 million,

I will sell you a tool that enables you to keep your codebase healthy and your developers productive forever.

And if everyone chips in,

I will even grant you exclusive access to it,

and you can resell it however you like."

This would be an amazing deal, right?

Like, you all stand to make a massive pile of money,

and I get to go retire to a beach somewhere.

And yet, no matter how many times I make this pitch,

I never see anyone reach for their checkbook.

Even if it was a serious business proposition, nobody's going to buy that.

We all have a deep understanding that such a thing just can't exist.

It must be snake oil.

And why is that?

And the big hint here goes all the way back to the 1970s and

'80s with Fred Brooks and The Mythical Man-Month essays.

There is no silver bullet.

There is no one tool or language or engineering practice that is going to provide a 10x productivity boost.

Brooks originally phrased this as,

"We won't see a 10x productivity boost in 10 years from a single source."

But I think it's broadly just true.

Like, it's not going to be one thing.

Instead, it's a continuous slow march as we refine our technology,

processes, ecosystem, and even the understanding of the nature of the problem.

We saw that with the site reliability engineering books, the the SRE book.

It isn't just one thing that takes you from the bad old days of spotty uptime,

low reliability, early internet,

and gives you Google-style reliability and performance.

Instead, there are host of changes that are necessary.

You need to make changes in culture.

You shift from "Who let the site go down?"

to running blame-free postmortems and figuring out what set of technical affordances and triggers led to that outage.

And then you resolve those rather than just fire whoever is uh unable to dodge.

Uh you need changes in technology.

All right, the SREs will always tell you you need monitoring if you want reliable production services.

And even in how you conceptualize the problem in terms of error budgets.

And that one I think is particularly enlightening.

Rather than saying that the goal is 100% uptime,

which is difficult, expensive,

causes everyone to be extraordinarily risk-averse,

and usually can't be noticed by users who don't have 100% connection to the internet,

instead, we set a goal of 99.9 or 99.999% uptime.

And then we use that budget of downtime as a resource to adjust our risk tolerance.

If you've already spent the budget for the quarter,

then you need to be very slow and careful with your releases.

But if you haven't spent it yet,

you can focus more aggressively on new features and experimentation.

That change of a tenth or

a hundredth or a thousandth of a percent in the goal changes the entire understanding and

framing and management of the problem.

So, taking a cue from the SRE book and all of that SRE thinking,

I'd like to present our own core tenants,

the ways that we are framing problems that I think help make software

engineering as we understand it at Google different and effective.

These are the main themes and theses of the Flamingo book,

time and scale and trade-offs.

These very heavily overlap with software engineering definitions and

thinking that go all the way back to the earliest days of the term.

Like Dave Parnas or someone in that crowd as the source of the quote seems disputed said all the way back in the 1970s,

"Software engineering is the multi-person construction of multi-version programs."

And I think that this is a really prescient sort of definition.

Multi-person means we're talking about teamwork.

Multi-version means we're talking about time.

And we're going to start today with talking about time.

How does time affect our projects?

The question that I'd love to ask is what's the expected lifespan of this code?

It's important to realize that there are roughly six orders of magnitude of a reasonable answer to this question.

That is, I could be hacking on something that I intend to throw away in 5 minutes after I've,

you know, written my Python script to process some logs,

or it could be code that I expect to last for decades.

And yet this question is rarely discussed.

Very often when a novice is learning to program,

the lifespan of the resulting code is really practically measured in hours or days.

Those programs, those code snippets,

are generally not touched again after their initial production.

In some colleges, you might have a project course or a hands-on thesis.

For most students, that is going to be the only time that their code is likely to live for longer than a month or

so.

That short life expectancy for code doesn't necessarily change immediately upon entering the industry.

A large amount of the software industry is focused on startups or app development,

and both of those have a notoriously short expected lifespan.

Startups are between 1 and 3 years, depending on which stats you look at.

The turnover on apps and app stores seems to be about 2 or 3 years as well.

Most of the effort at a startup focuses,

by necessity, on keeping the business alive.

The company may not live long enough to reap the benefits of an investment

in long-term sustainability for your software.

And then, of course, even that companies and organizations that are going to live,

you know, decades, there are still a lot of projects that are just

thrown away after a couple of years once they get too old.

But, by counterexample, successful industry projects,

or even open-source projects,

may have an effectively unbounded lifespan.

We can't reasonably predict an end point for the Linux kernel or the Apache web server.

In most Google projects, we also can't predict when we will no longer need to upgrade our dependencies,

language versions, et cetera.

We can't predict at this point whether there's going to be a security

vulnerability that requires us to drop everything and rebuild, redeploy, make changes.

These long-lived projects have a very different feel to them than programming assignments or startup development.

Another way to look at it,

on the short end of the spectrum,

if I'm just hacking on that Python script and a new version of my operating system comes out,

should I stop what I'm doing and make sure my script works in the new OS?

No, that would be bananas.

You're in a flow state working on a task.

Finish that.

It's extremely unlikely that the OS upgrade is that pressing.

But, on the other end of the spectrum,

if I have code that was written at Google 20 years ago,

and we still haven't been able to update it to the current operating system or current version of its language and libraries,

that's pretty clearly a signal that something is deeply wrong.

And together, even just these two data points mean that there is,

somewhere in this time frame,

a really scary transition.

I estimate somewhere between the five and maybe 10-year mark,

you start getting into, "Yeah,

this really has to be upgraded."

And that's also, unfortunately,

right about the outside edge of the experience level for many developers.

Many of us have not worked on the same project for longer than that,

sort of five-ish year mark.

And unfortunately, the cost of paying down everything that has built

up in those first years is particularly rotten for three main reasons,

which all, I think, sort of multiply together.

The first is that you have to overcome what we've taken to calling Hyrum's Law.

And I will make a note,

since Hyrum is moderating here,

he did not name this after himself.

He just made the observation.

I'm the one responsible for pinning his name on this,

and I take great pleasure in seeing him squirm about it.

Anyway, Hyrum originally had a sort of throwaway observation here.

He was one of our first engineers to master what we've taken to calling large-scale change refactoring,

the tools and policies that allow us to refactor interfaces that are used by tens or

hundreds of thousands of callers.

And Hyrum saw that with a sufficient number of users of your interface,

it does not matter what you promise the behavior is.

All of the observable behaviors of your system will be depended on by someone.

It turns out that when you see at scale the frequency that things just happen to work,

rather than being correct like by contractual like obligation and reasoning,

it flips your justifications a little bit.

You can wish that people were more disciplined,

but at at some point, the evidence is overwhelming,

like, you're wishing for nothing, right?

It's not going to play out that way.

We need to just take this into account and plan for it.

So, these days in discussions of design and refactoring,

we treat Hyrum's Law as a sort of thermodynamic truth in software maintenance design refactoring projects.

It's like entropy.

You can never entirely win, but you can improve efficiency.

You have to be aware of it.

You have to respect for it.

You have to plan for it.

For instance, one of the major remediations that we have found against Hyrum's Law is non-determinism.

If your users cannot reliably depend on your behavior, then they won't.

So, if you don't want to promise a particular behavior,

you mix it up a little bit.

This comes up in all sorts of places, but the easiest to describe is hashing.

Countless subtle dependencies exist on the current hash behavior in most programs.

Changing the hash function,

changing default allocation sizes,

changing anything about your hashing strategy is going to change the order that elements are returned when

you iterate over a hash-based container.

And that's a drag because hashing is central to efficiency,

especially in large systems.

But, the constant march of technology also means that there are new approaches to hashing,

hash functions, container layout,

all of these things, and

you won't be able to deploy those improvements if people depend on the old hash iteration order.

So, instead, languages and libraries may come up with ways to add some non-determinism to prevent that.

Randomness is a mitigation for Hyrum's Law, but it can only be a mitigation.

Just like entropy, you can't actually win.

We saw this with the Go programming language.

Hashing has been randomized in Go for quite some time,

and that gives the Go team the capability of making changes to the container for performance reasons or

even just to clean up the code.

But, we have also found people depend on that behavior.

One poor way to shuffle a sequence is to throw all of the elements into a randomized hash and

then iterate through it.

Turning off the randomization,

turning off the Hyrum's Law mitigation,

will break the users who have come to depend on it.

This is what I mean.

You can't win.

You can't even break even.

You have to be aware of it.

You have to plan for it, but you're never going to win.

So, the first major point in why are upgrades hard,

and especially after the first upgrade after years of stability,

is that you have to shake out all of these sort of hidden dependencies.

And that can be anything and everything.

If you haven't changed anything in your dependency, your project structure, etc.

for 5 or more years, then at scale you're going to find all sorts of nasty things lurking.

But, we do find that these build up slowly and consistently.

If you have sort of shake it up and keep shaking,

it is very possible to keep all of this at bay.

The second major difficulty is that you're treading new ground.

By definition, you're doing a thing that hasn't been done to this project before.

You won't have policy in place for how to decide what is safe.

You won't have experience on the team to fall back on.

You're setting new precedent.

And that's always going to be hard, both technically and organizationally.

And lastly, we tend to say, "Oh, hey, we're upgrading now."

And so let's go all the way to whatever is current.

So, a long deferred upgrade tends to be we're going from version 1 to version 5,

instead of the nice incremental going from version 5 to version 6.

That's more change to absorb, and inherently therefore also hard.

I think each of these individually would make a task like an upgrade challenging,

but they're not independent.

They they each make the others more difficult.

And so, the multiplicative difficulty here is very, very challenging.

And thus, after actually doing an upgrade for your project once,

or giving up partway through,

it's very reasonable to overestimate the cost of doing the next one,

and you decide never again,

and you commit to always just throwing things out and rewriting.

This may ring true to many in the audience, I think.

Getting through not only that first process,

but getting to the point where you can reliably stay current going forward,

is a big part of what we call sustainability.

For the expected lifespan of your code,

you have to be able to change all of the things that you ought to change and safely.

And for particularly long-lived projects,

making that transition to sustainability can be awfully challenging.

But this phrasing is interesting specifically because if the expected lifespan of your code is very short,

you're sustainable from jump, all right?

If I know that I'm going to throw this away in 10 minutes or a day,

there's very little around me that I'm going to have to react to change.

Uh I don't need to deal with an OS upgrade.

I don't need to deal with a language change.

I don't need to deal with security vulnerability when I'm just going to throw it away at the end of the day.

Whereas it's a entirely different reasoning,

it's entirely different problem domain when we're working with 5-year and 10-year and 20-year code.

So when someone like Google says this is how we do these things and why,

I completely understand that a chunk of this sounds like utter nonsense.

Why do we have all of these extra steps and safeties?

It's never going to be necessary.

It's never going to pay off.

But what I'm saying is when you're on a long-lived and sustainable software ecosystem,

you're playing a different game with different best practices than

you would be in a more short-lived programming project.

I think that a lot of these practices like testing,

code review, and those sorts of things actually do pay off in the short term and will serve you if you grow.

But I'm only actually going to fight you on it if you're working on long-lived code.

Software engineering thus is not just programming.

It's the art of making your programs,

making your code, making your project your code base resilient to change over time.

In the same way that SRE is pushing people to realize that error budgets are a better definition of the problem,

software engineering is about realizing that the production of software is not the goal.

Solving problems and keeping those solutions working for as long as is necessary is the goal.

You happen to do that by writing some code, but that's just a means to an end.

It's not the goal itself.

This is the first pillar in our understanding of software engineering, right?

Be mindful of time.

There are hard problems in programming to be sure,

but they're largely challenges in execution, right?

Writing bug-free code, understanding the problem that you're solving.

It isn't until you add time to the mix that you start running into the all of the problems that are really nasty.

Version skew, merge conflicts,

race conditions, schema evolution, backward compatibility.

So much of the really subtle stuff that we fight with has one underpinning commonality.

Time.

Time is the hard part.

Time is the extra dimensionality that takes our nice,

understandable programming problems and makes them something truly nasty.

Therefore, I say software engineering is programming integrated over time.

If you get to the point you can successfully make things last,

then the next thing that you need to probably worry about is growth.

When change over time leads you to growth,

your your team is expanding,

your company is expanding, your project is growing.

Where do you start to fail?

Are the things that must be done repeatedly scalable?

So, for instance, if your project doubles in scope and

you have to perform an expensive task like an upgrade a second time,

is it going to be twice as labor intensive because you have twice as much code?

Is it four times because you have a communication scaling problem in there?

Or is it cheaper because you have practiced and automated and refined your ability to do this?

I'm just going to say everything that your organization relies upon to produce and

maintain code existentially needs to be scalable in terms of general cost and resource consumption.

In particular, it needs to be scalable in terms of human input.

No task in your workflow,

no task in keeping your organization running,

should require increasing heroics or super-linear communication scaling as the team or code base grows.

No task should require broadcast messaging.

Those are not going to work as you get too large.

And importantly, there are going to be lots of tasks that can be solved by those approaches,

and they may even work out okay early on,

but the issue is there will come a point where they're holding you back,

and you have to track them,

be aware of them, be ready to get away from them.

Time and scale tend to play together in a lot of ways.

Uh so, we'll consider what Google has learned about deprecation and churn many years ago.

The idea with many with a lot of traditional deprecation is I have an old thing, I don't like it anymore.

I have a new one, I want everyone to use that instead.

So, one way to do this is to just ignore the existence of the old thing, right?

You mark the old one deprecated,

you introduce the new one, you just call it good.

But, over time, as you do this repeatedly,

as new versions are introduced,

if you don't actually have a forcing function to push old unmotivated users off of the old thing,

to break the dependencies from old unmaintained code,

for instance, you're going to wind up with an ever-increasing amount of maintenance for purely legacy systems.

And sometimes that doesn't matter, right?

Having two versions of a function isn't actually a lot of maintenance burden for the library maintainer.

It's okay to have foo and foo2 and foo3 for functions.

It's not great, but you're not going to have a really solid argument for why it's terrible.

But, higher-level interfaces do actually have increasing costs,

both for the maintainer and the environment around it.

Uh types, especially types that pass through interfaces,

storage systems, microservices.

All of these things are much more costly and nefarious to deal with having duplicates than just a function would be.

Yes, it may cost you something up front to push developers off of those old systems,

off of those old interfaces,

but that one-time cost might be less than ongoing legacy maintenance,

especially when ongoing might mean indefinite.

Another approach is you force people off of the old thing by scheduling to delete it or turn it down.

But this mandating approach doesn't scale, either.

Imagine as your code base expands, right?

As you have a graph of dependencies among all of your projects,

the leaf projects have an ever-growing set of dependencies below them.

You can look at any, you know,

graph growth algorithms and and studies, those sorts of things.

If every dependency below you has the right to say,

"All users above us need to make this one change to move from the old thing to the new one,"

then you're increasingly forcing all of those leaf teams to run just to keep up.

Or worse, they don't.

And annoyingly, each individual on those teams that that work gets

foisted off on is stuck learning the details of the task, right?

As a large organization, we pay for that ramp-up cost over and over and over again.

And we when we do it to our external customers, it's even worse.

Worst of all, in honesty,

most projects that depend on whatever interface you're providing aren't actually leaves.

Other things depend on them as well.

And that means that the threat itself is sort of disproportionate,

especially in a continuous development or trunk-based or mono repo sort of world.

You aren't just breaking the build of the project that didn't obey your threat,

you're breaking everyone downstream of them.

Imagine my real-world a little hyperbolic analogy.

Timmy, if you don't put away the dishes, I'm going to blow up the city.

This might get the dishes put away,

but if it doesn't, then either your threat was false,

or you're primarily harming bystanders that had nothing to do with the great dish debacle.

Policies about maintenance or upgrades,

or really anything that are pinned on threatening to harm bystanders,

probably not the right design for your policies.

You could also resort to heroics,

just have every have one person fix everything all in one go.

Heroics don't scale either.

This particularly doesn't work if you don't actually have perfect visibility into all of your users.

If you don't have a mono repo or the logical equivalent,

then this sort of thing is out of the question.

Instead, at Google, we hold to the churn rule.

The majority of the cost of any change must be paid by the team that is making the change.

There needs to be a published and reviewed sort of transition plan.

This is going to highly constrain designs for the new thing,

because you have to have enough semantic overlap that you can find a path from where you are to where you want to be,

but it does put all of your incentive structures into much more scalable places.

And buried in this is a very important point,

which is practically speaking,

policies like this scale better specifically because

knowledge and expertise within a team within an organization have very good payoff,

often with sort of superlinear effects.

That is, when you make the team that triggers the change into specialists that understand how to execute the change,

it's a much, much cheaper outcome in a world,

at least where all of your users are visible to you.

You're only going to pay the ramp-up cost of learning the nature of the problem a couple times,

and then expertise and automation can make things much quicker and cheaper overall.

In a successful organization,

everything that must be done repeatedly must consume sublinear resources,

especially sublinear human effort and communication.

This also applies to code base size or build and test resources.

The growth rates on all of these types of problems tends to be slow but steady.

Monitoring your build times,

your resource consumption is a good idea to help prevent boiled frog sorts of problems.

Monitoring your human processes for things that are becoming all consuming

for some poor heroic soul is even more important.

Superlinear scaling for any cost is a serious problem for your organization.

No surprise there.

But on the flip side, those same properties of scale that make those

costs dangerous make the hunt for scale-based efficiency extra potent.

In a well-functioning organization,

lots of tasks, especially those that probably don't become necessary until you've been around for a while,

can be carved off for experts.

Compiler upgrades, OS and kernel updates, API migrations.

The impact of those experts tends to be superlinear because of accrued expertise, automation, familiarity.

You can just manage more things when you specialize.

When it comes down to it, you have to be aware of scale.

Don't fear it, respect it.

It's a sign of success, after all.

For another glimpse into how does this scale,

consider tactically uh version control policies and merging.

It isn't rare to encounter shops that have very strict policies about who gets to merge into trunk and when.

And it's not even hard to understand why.

A team could be off developing on a branch for a month,

they finish, they merge, and then you find that the release is unstable.

Tests are no longer passing.

We've all seen this before, right?

Big merges can be costly.

And when you identify a thing that has two options,

you have or that that is is risky or costly,

those sorts of things, you have two options to handle it.

You control the cost, you lock it down, or you just make it cheap.

Right?

Practice, reduce the cost, make it common.

Depending on the situation,

either of those could be good,

but for version control in particular,

the control the cost, make this happen rarely path is just bad,

and the research shows this.

Uh the trunk-based development leads to better outcomes research coming

out of uh groups like uh the DORA uh State of DevOps reports,

the book Accelerate, all of these things.

Uh it says a ton on this topic.

And I think that this is a perfect microcosm of what I mean when I say,

"Watch out for scaling problems."

Companies that I was involved with before Google absolutely relied on

this sort of merge strategy meeting with different teams and

different contractors working on long-lived dev branches and deciding who gets to merge when.

At one point, we even had to hire a second content management engineer to handle the load for doing merge and

retest.

Because when we got to 200 engineers on the project,

the guy that was in charge of the merge before couldn't keep up.

Adding a second engineer didn't double the throughput on the number of merges we could do because

those two people had communication and coordination overhead,

just as would be unpredicted in every engineering text for the last 50 years.

By comparison, Google has a couple hundred times that many full-time engineers working on one code base.

Nobody is responsible for the merge.

It all just works.

And it isn't that our engineers are that much better,

it's that we spotted that the policy of long-lived dev branches and coordinating how we merge wouldn't scale.

So, we found something that required fewer decisions,

less communication, less coordination.

Another example of time and scale playing together is shifting left.

In the past decade or so,

this idea has emerged that has become a really common piece of vocabulary in software engineering or

DevOps methodology.

Shifting left makes things cheaper.

The earlier in the software workflow that you can catch a bug, the cheaper it is.

And I actually see this as a direct result of thinking about systems in terms of time and scale.

Originally, this was phrased as shift left on security.

Right?

If you remember in the late '90s and early 2000s,

as an industry, it was very common to see,

"And I guess we'll think about security."

tacked onto the very end of the software process.

And as it turns out, that's very expensive.

Software that has already been released is constrained.

It's hard to change.

It's depended upon.

There are Hyrum's Law problems.

Security bugs in the wild have legitimate real-world consequences,

potentially up to the loss of human life.

Catching those same bugs before that software is released into the wild is much cheaper.

Users aren't depending on the feature.

You don't have the same legal liabilities.

We can fix it and try again.

Paying for testing and and maybe canary release costs.

But that's still much cheaper than a full release process or catching the bug in production.

It would be cheaper still if we caught it in code review.

At that point, the developer responsible knows what they were trying to do.

They already have the details fresh in their head.

Languages like Rust are prioritizing making it hard to even express the riskier concepts,

pusing the focus on security all the way back into the development step.

The same basic principle of shifting left applies to bugs of all forms, not just security.

If a bug shows up in canary, well, that's better than it making it to release.

But now we have to look through all of the changes that happened since

the last release to see what actually broke it.

If we catch it pre-submit, you know exactly what broke.

If you catch it during development,

it may not even register as a hurdle during the developer's work.

We add steps to the workflow,

usually, in order to catch defects earlier and cheaper.

But you have to be aware and admit to yourself that the purpose of those steps is not 100% defect reduction.

That's impossible.

Bugs will always slip through.

The only high fidelity indication of being defect free or at least defect neutral is to release it to your users.

Run it in production.

And the costs and risks of doing that are extremely high.

So, you add all of these proxies trying to balance between cost and

fidelity properly for each possible class of defects.

As I said, I think that this ties to time and scale both.

It's a time issue.

Imagine building two houses.

One construction company has alarms in place that detect if

something was done in an unsafe fashion and sounds the alarm instantly.

The other company has detection that is just as good,

but the alarms only go off on move-in day.

Which house would you buy?

Heck, which house would you rather build?

But, it's also a scale issue.

If we handle time perfectly by mushing moving to push on green continuous deployment and

we can submit a change and almost instantly see it in production if the test pass,

it's still cheaper to find bugs earlier in the process because

you don't have to worry about was this caused by me or

someone else or the interaction of our two changes or

something unrelated in just how our software is scheduled and deployed.

Scale here is the second important pillar when

considering tools and policies and practices in software engineering or really anything,

but it is particularly acute here.

Considering the idea of time,

how things are going to change,

and scale, how your team and your organization,

your workflow are going to work as you grow,

all that's left is make good choices.

Make evidence-based decisions.

If you understand that change must be possible,

if you're explicit about the lifetime of the software that you're producing and maintaining,

and you understand that the scaling goals of your organization,

your engineering process, all that's left is to focus on making good decisions.

People often say that everything in engineering is about evaluating your trade-offs,

and we find that that is true even in software.

The important thing is to make data-driven or at least evidence-driven decisions,

accounting for the realities and the requirements of time and scale.

You have to aim for sustainability, right?

You have to aim for that state where you are capable of reacting to every change that comes along.

You can choose not to.

That is a potentially a reasonable description of technical debt.

But you have if you can't actually react to changes,

then you are making a very high-cost,

high-stakes bet that nothing important is going to need to change.

And in a world with Log4j and Spectre and Meltdown and Heartbleed and,

you know, the regular every year or two cadence of internet-breaking security bugs,

that seems risky to bet that nothing's going to have to change.

Change must be possible.

No superlinear scaling, especially for humans.

Be mindful of your processes and your communication.

And critically, because time and scale both have a tendency to change things,

you're going to have to re-evaluate.

Context changes.

The reasons, the evidence that go into any given decision will also change over time.

A decision from last year may have been right given the information that you had available then,

but it may become wrong next year.

It's reasonable to summarize all of this as make logical trade-offs.

In many respects, that shouldn't need to be said.

Of course, you need to make logical choices if you want good outcomes.

This is less about a surprising insight and more about focusing on the important philosophy.

Within Google, there's a strong distaste for because I said so.

Inherent in all of this is the idea that there needs to be a reason for things.

Just because or because I said so or because everyone else does it this way,

those are the places where your worst decisions are going to hide.

So, what's the secret?

I think it's these three things.

I think it's time and scale and trade-offs.

It's software engineering is the multi-person construction of multi-version programs.

You have to be aware of the impact of time and the expected lifespan of the software that you're uh managing.

You have to be aware of scale and the communication overheads,

the scaling properties of the human algorithms that you're running.

Uh superlinear scaling is very very bad for your required processes.

But, on the flip side, expertise and specialist can take that scale problem and make it into a benefit.

So, there is some upside there.

And you need to make evidence-based decisions.

Don't rely on because I said so.

And remember that the evidence is going to change over time.

So, you have to re-evaluate as needed.

It's not a sign of weakness to admit that your reasoning has changed or your thinking has grown.

The Software Engineering at Google book as a whole,

this talk is basically chapter one.

They lay out these pillars and

then throughout the rest of the book in every chapter we tried our best to tie back to these ideas of time and

scale and what things were actually trading off.

We start the book with a extensive section on culture,

like how to be a good team member,

how to design inclusively,

how to lead an organization,

how to uh grow your organization.

Uh we also look at policies and and uh processes,

the sort of human algorithms that make your organization work smoothly.

And then we get into a little bit into the specifics of tech,

the tools that we deploy in all of this.

I will leave you with the idea,

it's programming if clever is a compliment and it's software engineering when clever is an accusation.

And if you understand just how deep that truth holds,

then you might be a software engineer.

All right, thank you Titus.

So, as a reminder, please ask your questions in the question section and

I will launch a new poll here that have come in while we've been while we've been discussing.

So, uh there's been a couple of questions about dependencies and

and I'm going to not going to say names because I will probably not pronounce them correctly,

so pardon me for that, but

um the one of them is you think it's amazing book in your chapter

about dependency management you admit that you don't have established best practices quite yet,

but rather solutions that are less bad than what came before.

Do you have any new learnings or insights into the complexities of dependency management at scale over time?

No.

Basically, no.

And I think a lot of it really boils down to I think it's irreducibly hard in some respect because

the the difference between a version control problem of like where do I check this in,

what version of this do I depend on,

uh and a dependency management problem is uh at least in in the definitions that we used in the book,

when you're dealing with dependency management,

someone outside of your team,

someone outside of your organization is producing that other thing.

Right?

They may not have the same priorities that you do.

They may not have the same goals in terms of efficiency or stability that you do.

And the more that you build your software out of pieces that are controlled by people with different priorities,

the harder it's going to be to make sense of that.

I I that's inherently the problem.

And this is actually another point where this distinction between it's a programming problem or

a software engineering problem really shines through.

If it's a programming problem,

grab every decent quality,

like not security hijacked dependency you can find because that infrastructure is going to like let you express your problem,

solve your problem much more efficiently than writing it all yourself.

But when you get into have to manage this for a longer and longer period of time,

you have no idea if the maintainers of those other dependencies,

be they open source or paid,

are going to be interested in 5 years or 10 years.

You have no idea if they're going to stay up on security bugs.

You just have no idea what their priorities are going to be.

And that is a realization and

a and a set piece of complexity we just don't have a good answer to because

I fundamentally don't think it's a technical problem at its root.

It's a matter of conflicting priorities and trying to balance those things.

It's hard.

It's always going to be hard.

Yeah, there's a there's a lot of human factors in in this stuff.

Uh You know, I I I tell people within Google that like if if it was not a human factor problem,

we would write a computer program and it would do it.

The ergo, all the rest of the problems are human factor problems because

we haven't programmed the computer to solve the problem yet.

Yes, exactly.

Yeah.

So, um there's been a couple of uh questions about tooling and you know,

at Google we have lots of specialized tools for a lot of these things.

Um you know, how do companies that are smaller use some of these principles and

use some of these tools even if they don't have access to the specialized tooling that that Google may have um you know,

and and and that kind of thing.

Uh I I don't have a perfect answer to that, of course.

Um but I think, weirdly,

and I'm going to take a kind of strange swing at it,

uh I think the most important thing to get right from jump is have a good build system.

Because it turns out in my mind that a lot of our problems with

tooling really come down to uh build system integration and

like understanding what the code is actually doing and being able to deploy the tools consistently.

And among other things, uh having the delta between this build system works and

this build system is actually good is kind of surprisingly deep.

And most people I I always say,

if you ever have to run make clean,

your build system is bad.

And people kind of look at me weird.

Uh but if you get to the point where your build is distributable and cacheable and parallelizable and all of those things,

you can use the compute resources on other,

you know, people on your teams' machines, those sorts of things.

Uh that level of like consistency and understanding in what your software,

how your software is built uh is unfortunately the blocker for so much of like tool application,

consistent tool usage, like better habits and patterns.

And again, that's also challenging because

a lot of our uh external dependencies are on build systems that sort of happen to work,

not build systems that are inherently well designed.

And the the chapter on build systems in the book,

I did not write, but was one of the most eye-opening things for me to read.

Um that so.

Yeah, it was and and it it that highlighted the constraints.

Like having well-defined constraints in a system actually is really really useful from an engineering perspective.

That was actually an emergent property of the book as as chapters came in.

We didn't write all the chapters.

We had people contribute them.

And as they came in, a number of them touched on if

you have well-defined constraints that actually makes your engineering process easier rather than harder.

I mean, all the engineers like to turn all the buttons and

press all the knobs and or press all the buttons and turn all the knobs and do everything,

but like constraining what they can do actually makes your system much more scalable.

Yeah.

And it it's it's one of the It's one of the really difficult tensions, right?

Because as a as a provider of software,

widget, API, interface, whatever,

uh you want to support every request from your users.

You want to support every use case uh because that feels like good,

you know, product and customer focus.

But the more that you support those things, the less consistency there is.

And the harder it is to make future changes or to even reason about how users are using your system.

And I don't think we talk about that tension enough.

Yeah.

Absolutely, yeah.

There There have been a couple of questions uh come in about time

in the context of the evolution of software systems.

So, often when we build a system,

um we don't expect it to last as long as it may end up lasting.

And so, uh you know, answering this question,

you know, how long is it going to last ahead of time um isn't always uh the right an- you know,

isn't always a true answer as time progresses.

And so, I wonder what thoughts you have,

you know, how to address that in in kind of the software process.

Or what what happens when software ends up lasting years when you only intended it for days, kind of thing.

Right.

Uh so, I I'm increasingly kind of coming around to maybe a three-tiered

model of there is the stuff that is really just quick throwaway whatever.

Like I hack it together in you know, 5 minutes, an hour, something like that.

And for those things, like regardless of if it let like if I run it again once a year,

it's not really that much investment.

Like if something goes wrong,

if it's not up to date,

if it bit rotted for some reason,

then I could just write it again, right?

It's a 10-minute or hour investment of time.

I think that there's a distinct additional class that comes in at the like I this is an experiment.

I intend to decide is this going to survive in 6 months or a year?

Uh those will be better in general if you actually delete them when the experiment fails.

Uh and then if you don't really have a sense of why this code is going to stop being useful or

stop being used uh after a year or two,

then you probably might want to start taking uh a longer-term look at things.

And if you're in a startup environment or something like that,

I it I'm not even necessarily lobbying that you need to change your process ahead of time, right?

You you still have that existential risk of making the next funding round.

But you know, start keeping a list of the things that this is kind of a hack or these haven't been updated or yeah,

this policy isn't going to actually work,

uh so that, you know, the next time that you land that funding round,

you can start, you know, making plans.

Uh because the the misery of it is a lot of these sorts of oh, hey, I need to upgrade.

Oh, hey, I need to be able to fix these problems.

They very rare very regularly turn out to be important but not urgent.

And I think a lot of what we're actually fighting as software engineers is how do we convince ourselves,

trick ourselves, may keep awareness of the important but

not urgent set of tasks and and carve out time to time and energy uh to to pay down some of those things.

Uh there's a specific question here that I want to read but I think it highlights a broader area.

So, I'll just read the question and then kind of um bend a little bit.

So, hardware is cheap is something that is used a lot to justify designs that seem to be poorly designed.

Do you think the overheads on managing and

distributing managing distributed and

hardware scaling is under appreciated when

doing design or just throwing more hardware hardware at the problem maybe the best first instinct.

Uh and and I think that that question highlights more a more broad set of things around technical debt and

making trade-offs in in designing systems and you know,

how how how do you you know,

how how do we use the or

how do we best use technical debt or

is technical debt always wrong or like how does that play into the the engineering um design process?

Okay, so I will first I will talk about the resources thing.

I will riff a little bit on technical debt and

then I'm going to actually kick that back to you cuz it's your day job.

Um but the the resources thing,

one of the one of the uh one of the resources that we have internally

that I absolutely love is we have sort of a consensus trade-off on

in the abstract what is 1 hour of engineer effort worth in terms of CPU usage or

memory or network bandwidth or disk storage or you know,

any of the compute sort of resources.

And having even if that's not a perfectly accurate sort of set of trade-offs because,

you know, price fluctuation and,

you know, some engineers are paid more than others, all of those things.

But like having a baseline consensus for this is how you do the math to decide whether or not,

you know, these particular trade-offs are worth it.

Uh, is a really important you know,

like counterpoint to well, hardware is cheap.

Right?

Uh, the the well, hardware is cheap response is can cover up so much of uh,

well, because I said so or because we don't know or because well,

I just don't want to think about it.

Right?

Whereas if you actually get down to just do a back of the envelope calculation of like, okay.

Uh, we can spend another day on this design.

Uh, we currently think that it's going to cost this much to run it

in production for 100 users, 1,000 users, whatever.

Uh, does that back the envelope math justify it?

Right?

And when you make that trade-off clear for your organization and like everyone can just use that math as a as a,

you know, back of the envelope calculation,

that takes a whole lot of the sort of bombast and uh,

arguing from the hip uh, out of the equation.

And I think that's really important.

Uh, when that veers off into technical debt,

uh, you know, all I'm actually going to say is uh,

boy, I hate that we anchored on that term.

Uh, cuz I I really don't like the finance metaphor of it.

You and I know it is much much more challenging to fix things after the fact because people depend on it.

There's a law about this.

Uh, and uh, the the because

there is such a disconnect between the cost of it never happened in

the first place versus the cost of cleaning it up afterwards,

I am much more of a fan of like a pollution or just making a mess metaphor instead of a finance one.

All right, cuz then the finance metaphor,

the badness of one, you know,

unit of debt is one unit.

And it costs one unit of effort of goodness to pay that off.

And those costs are not actually symmetric in practice when you're when you're talking about larger technical debts,

but but I think that there was something you were trying to get at other than maybe leading me to really.

I didn't have any There was no ulterior motive in asking that question.

I will agree though I will agree.

I will agree that you know,

people often underestimate vastly the amount of effort it takes to clean something up because

the thing exists and you have users and customers and being able to you know,

you have to as long as you don't want to alienate your entire customer base,

it is really really difficult to keep a thing running and clean it up at the same time.

Like clean out the internals at the same time.

Um kind of related to that, there is a Hyrum's Law question.

So, I want to have this one and then we'll probably have time for one more after this.

Um And and the Hyrum's Law question is about you know,

if the API consumer depends on the Azure Azure behavior not dependent on this not specified in the documentation,

isn't the onus on the consumer to make changes to their client code when the behavior changes?

Um you know, or or and relatedly,

like if a team you know,

makes a change that triggers a lot of work,

uh how can they do that work if it's in code bases or other environments that they're unfamiliar with?

Right.

Uh so, this sort of philosophically is not actually a talk a question about refactoring,

it's a question about visibility,

mono repos, version control versus dependency management.

Right?

Uh my my absolute guidance is prefer version control problems over dependency management ones.

Right?

If you can have uh all of your code visible and coordinated,

if you have one build, one CI system that can tell you,

yes, the build is currently working,

uh that is vastly preferable.

The other common pattern that we see is every team,

every project has its completely separate repo,

and you can't see any of your users.

And a lot of this boils down to our like People talk about backwards

compatibility being a property of a change, and that's nonsense.

Right?

Like, I am not a compatible husband.

I am compatible with my wife.

Right?

Compatibility is a property of a relationship, not of an entity.

Right?

And so, like, at most you can say,

I made changes to this in a fashion that I believe to be backwards

compatible under the set of uses that I intend to support,

but because of Hyrum's law,

you can say that as much as you want,

and it's not going to actually matter all that much because if you have enough users,

someone was depending on the thing that you decided to change.

And so, when you look at the the cost of all of this,

uh yes, it is cheaper for me to make a change blindly without seeing its impact on my users,

the broader ecosystem, the whole organization,

but it is cheaper for the organization if

I don't make that change until everything around it is compatible with that change.

And yeah, that's more work.

It's harder, for sure.

But the way that you tackle it is specifically by having visibility into as much of your users' code as you can.

Like so within the bounds of one company,

have one clear shared understanding of what's checked in.

Like what's depending on what and at what versions.

It doesn't have to be mono repo mono repo, but needs to be kind of close to it.

Um and then two, it's really just a question of do you want one person or

one team to do all of the necessary work while they are automating,

finding common patterns, and learning the details,

or do you want the same amount of work to be done by hundreds of people,

all of who don't know what they're doing?

Right?

Within the bounds of one organization, that math is not hard, right?

That that is obvious.

Right?

It gets more subtle, of course,

when you're talking about things that are crossing repository or organizational boundaries.

Right?

But that is fundamentally a Okay,

now we're back to dependency management, right?

I don't have visibility into your code.

I don't control what you think is an acceptable usage.

I don't control what you think is the set of platforms that we need to support.

Right?

Like I think it all boils down to that.

So one final question, and I know a topic that's here close to you right now in the work that you're doing.

I would had a number of students ask questions about how to get more,

you know, how they can learn more about software engineering.

And aside from plugging the book,

um you know, and and a couple of professors have talked or people at universities that have asked about,

you know, CS software engineering specialized degrees and other kinds of things.

Um you know, what do you have a you want to take just a couple a minute or

90 seconds or so and talk about engineering education and what people can do and and and advice in that area?

Yeah.

Uh so, very quick, uh, we have made the English version of the book freely available.

It's available online and PDF.

Uh, you can search for it.

It's hosted on absale.io.

Uh, a lot of my projects go there.

Um, so that that might be part of it.

Uh, I'm sitting on the current,

uh, ACM IEEE AAAI, uh, committee to revise the computing curriculum guidelines.

Uh, so guidelines for groups of CS majors.

Uh, and specifically, uh,

I have a lot of authority over the software engineering requirements.

Uh, so the the educational material will probably change a bit in the next few years.

Uh, so I'm looking forward to that.

Uh, but broadly, it's inherently a hard problem uh,

to get this into school particularly because

if the dominant factors of the distinction between software engineering and programming are time and other people,

especially other people with varying degrees of experience and expertise and tenure,

age, all of these things,

uh, it's really hard to synthesize that in a undergraduate environment in particular.

Like it's hard to make problems that are dominated by the time nature or

hard to make assignments that are dominated by teamwork.

Uh, and so my hope is that a lot of software engineering like education especially for undergrads,

uh, focuses a a little bit more on like the the theories and

the research and the the ways of thinking about the problem that we're coming to.

Cuz this is all, you know, the whole discipline's 50 years old.

We're we're still learning this.

Um, but I I'm hopeful that we're we're going to shift it a little bit to have some project experience,

have some lectures about,

you know, the difference between version control strategies and why you use version control,

why you have tests, uh those sorts of things.

And and in practice, unfortunately,

I think a lot of it is going to be on the job.

I don't think there's a way around it.

Hopefully, we can help folks get a little bit uh easier into the the the job sphere.

So, thanks a lot, Titus.

Uh we've run out We're out of time for the day,

so I'd like to thank you again for your talk and the Q&A.

Um special thanks to all those that have participated,

and thank you for attending this this Tech Talk, this ACM Tech Talk.

Uh this talk was recorded,

and it'll be available online in a few days at learning.acm.org.

Uh you can find announcements on upcoming talks and other ACM ACM activities at learning.acm.org and acm.org.

Uh please fill out our quick survey,

which you can suggest future topics or speakers,

um which you should see on your screen.

And on behalf of the ACM,

Titus Winters, and myself,

Hiram Wright, uh thanks again for joining us,

and I hope you'll join us again in the future.

So, this concludes the talk.

Thanks, all.
