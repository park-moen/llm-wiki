# Fixing "AI Slop": How To Manage Agents Like MIT Interns w/ Jesse Vincent, creator of Superpowers

> Source: https://www.youtube.com/watch?v=aB2dVA-Y7CQ&list=LL
> Collected: 2026-08-16
> Published: Unknown

# Fixing "AI Slop": How To Manage Agents Like MIT Interns w/ Jesse Vincent, creator of Superpowers

K9 ended up with between three and 5 million daily activives as a hobby project.

Wow.

Somehow my life has become about reliable,

repeatable process and documentation and management.

And so I would come home at the end of the day feel like I hadn't done anything.

I had helped somebody with requirements.

I'd helped somebody else with debugging.

I talked to somebody else about their feelings.

I believe pretty fundamentally that the best way to become a engineer

is to become someone who can help 10 other engineers be effective.

But like if you hear that story,

it sounds an awful lot like working with Claude or working with Codeex.

When Cloud Code first came out,

first thing I did is I described something to it and

it went off and built this massive product that was insane.

It's asking you if if you're willing to use visual brainstorming,

which is a thing I added in Superpowers 5.

This is what it built for me.

This here is the entire Open Claw code base.

Uhhuh.

Uh visualized as a city.

And you can click on single buildings and it will show you then the code of that each building is one file.

What is superpowers and how is it built?

Sure.

So superpowers started off as Hello and welcome everyone to another episode of the merch,

our podcast here at the Code Rabbit office brought to you from San Francisco.

I'm Hrik.

I'm the developer advocate here and today I have a very special guest.

I actually a guest that I'm super excited about because

I've been diving very very deep into this very exciting word last night.

So deep that I almost lost track of time actually.

So with no further ado, Jesse Vincent, welcome to the show.

Hi, thanks so much for having me.

How are you doing today?

I'm doing well.

Uh Jesse, you've been building impactful technology for I think it's fair to say three decades.

Yeah, it is a little terrifying to realize that I have the actual gray hair now.

Yeah, I mean you look great.

Can you maybe walk us through,

you know, your career arc maybe for people who haven't heard about you yet?

Um, what is Jesse Vincent all about and what is superpowers and what are all your projects?

Sure.

So, um, when I was still in school, I started working on a ticketing system.

So, customer service, help desk, ops, uh, products called RT or request tracker.

It's open source.

It has been around for literally 30 years.

Um there's a little company I own back in Boston that still ma makes supports um and

you know and sells as as SAS RT but

it's all open source and

you know it's everywhere from government agencies to you know fortune pens the tiny little nonprofits.

It was it is still in Pearl.

Um and so that was so how I ended up for a couple years ending up as the project lead for Pearl 5.

So I was the first guy in the pearl and

it was always a guy for whatever reason the first guy running the

project who was more of a manager than our hardest core sea hacker.

So I was the one who had us finally write down a lot of our process.

I had a different person doing a development release every month and updating the process.

And so there was a whole lot of somehow my life has become about reliable repeatable process and

documentation and management and thinking about how to do things which is is what it is.

Um along the way, um I ended up creating an email client for Android called K9.

It was a fork of the Android 10 mail client because when Google shipped Android 10,

the non Gmail email client couldn't even connect to a server with a self-signed certificate.

Somehow along the way,

K9 ended up with between three and five million daily activives as a hobby project.

Wow.

When I had sort of had my fill,

I had handed it off to one of the folks who had been helping me as the new project lead and

I ran streaming to iOS to get away from it.

A couple years ago, K9 ended up becoming Thunderbird for Android.

It got my my my little Android mut got adopted.

It's kind of amazing.

Yes.

Um, for about a decade, my wife and I ran Keyboardio.

It's a tiny little open hard open hardware keyboard company making,

you know, f basically high-end ergonomic keyboards.

did put out four keyboards over the course of 10 years,

which is not a lot of product,

but is a two-person hardware company.

Um, you know, everything was funded on Kickstarter.

I spent a lot of time in China,

built a built an open keyboard firmware before,

you know, before that was cool.

Um, wow.

And yeah, that was a an interesting and fun experience.

During the pandemic, I was one of the folks running vaccinate CA helping

Californians figure out where we could get our shots because the government didn't know.

So, we had uh I think we were at 30 full-timers at one point and up to 300 volunteers calling every hospital,

pharmacy, and other distribution point in the state to try to find out if they had shots,

who, you know, who who could get a shot there,

what documentation you needed.

Um, and starting a little over a year ago,

um, actually about a year and a half ago,

I started playing with aentic dev right when everyone else did.

So, for a while that was flipping between cursor and windsurf week by week as they went through that arms race.

I remember that time.

Um, and then right around when CL when cloud code first came out,

the literally the first day I downloaded it and it was like, oh, this is the thing.

Mhm.

And the, you know, the first thing I did is I described something to it and

it went off and built this massive product that was insane.

It had all of these features that I hadn't really thought through or specified.

It was a full product.

It even kind of worked, but it was not at all what I wanted.

Um, and then I sort of stepped back and like,

how do I get this thing to behave?

And I started looking at like,

okay, who's done prompt actual prompt engineering and as in an engineering on prompts.

And so if you dig into my into my GitHub history,

there's a I think it's it's called like cloud setup docs,

which is a horrible repo name,

but it was I had this idea.

It took a very simple prompt,

which is let's make a React to-do list, use local storage.

And the first time I ran that,

it took 30 seconds and it cost a quarter and it built something looked kind of pretty.

But if you reloaded the page, it lost all the to-dos.

Um, over the course of probably a week or two,

I ran I ran a bunch of experiments,

basically figuring out what goes in a cloud.

MD.

And so for every experiment,

there's a snapshot of the cloud.MD,

the initial prompt, the transcript of opening up Claude and running that initial prompt,

so the whole session, and the output code.

And what I eventually got it to was a 30inut project,

a 30-inut fivephase project that cost $25 and did rigorous TDD,

including a failing test,

making sure that there's no packaged adj.

Um, which was, you know,

taking it a little too far,

but that was the core of me learning how you prompt a coding agent.

Um, and from there I sort of iterated on how do you get the agent to talk to you about requirements and

what you're actually trying to build and

then how do you get it to reliably engage in a reasonable set of

engineering practices to go do that building without getting distracted or turnurning out garbage.

And that's the core of what became superpowers, right?

um it does feel a little bit more like a philosophy more than just a dev tool.

Right.

So I mean so I think about it a lot like management.

So the first time that I did agent dev was probably about 2003.

Um I I had had this tiny little company making an open source ticketing system.

We were in Cambridge Mass and every summer I had MIT undergrad interns and I managed them on IRC.

So text chat window, you know, one line at the bottom thing scrolled past you.

Um, they were brilliant, they were friendly, they were agreeable.

Some of them thought they were God's gift to software engineering.

Many many of them, you know,

I mean, so many of them have gone on to be like amazing,

influential people who like and they were they were always amazing,

but some of them thought they were God's gift to software engineering as college freshmen,

as many college freshmen do.

They often did not yet have well-developed taste or judgment.

They were very enthusiastic.

They were clever.

They weren't sleeping super well,

which meant that they were having that they had trouble with memory formation.

And so I would come home at the end of the day as somebody who'd

been a working programmer feel like I hadn't done anything.

I had helped somebody with requirements.

I'd helped somebody else with debugging.

We'd talked I talked to somebody else about their feelings.

We, you know, I'd done some strategy work.

I done some planning but

I hadn't written any code and

the first time like that that was a really hard thing to get over for me and

but it was part of becoming a manager and

so that's also I believe pretty fundamentally that the best way to

become a 10x engineer is to become someone who can help 10 other engineers be effective right and

so but like if you hear that story it sounds an awful lot like working with claude or

working with codeex it like you know taste isn't necessarily so

great very enthusiastic virtuoso virtuoso developer who knows nothing about your project and

knows nothing about your requirements learning to work with that

like I learned how to work with that decades ago and

so when I first opened up claude it it felt very natural right and

it's basic I mean it's it is not a coding problem it's a management

problem is how you know how do you help this entity that it's not alive but it can think.

How do you help this entity that can think do the thing you want them to do?

And often that is you explain to them what you want and why.

You explain what you know what behaviors are good,

what behaviors are bad, and maybe there's some focus issues,

maybe they're a little overly literal, you can work with that.

Another thing is when things go off the rails,

stopping and saying, "Hang on,

you just did this thing that's kind of crazy.

What were you thinking?"

Not as an in an aggressive way, but like where are you coming from?

Why why did you think that was the right choice?

What could I have said that would have gotten you to do the right thing?

Mhm.

And a thing you can't do with a with a human engineer is you can but

you can do with a coding agent is double tap escape,

go back up to the beginning and change what you're saying.

Start the conversation over again and re and and re and replay having now said the right thing.

And so that's like one of the like some of the most powerful tools

that I have for working with agents are asking them why and then knowing that the cost to implement is almost zero.

And so there's no there is no this like the sunk cost fallacy is even more of a fallacy than it ever was.

You can just rewind and start over.

I think that's all a great leadup to superpowers, right?

Given us a lot of the backstory now.

So yeah, maybe give us the deep down brief.

What is superpowers and how is it built?

Sure.

So superpowers started off as a couple of prompts that I was using for here's how I make cloud do what I want.

Um I discovered when Anthropic first rolled out the ability to do Word docs and PowerPoint docs on cloud.ai,

I asked Cloud, hey, how are you doing this?

And it, you know, and it said,

well, there are these skill.mmd files in the opt directory.

Oh, could you give me a tarball of those?

And I read them and I sort of got these sense.

And so I built myself a skills framework for cloud code.

I didn't know that anthropic was also building a skills framework for cla code.

So I accidentally front ran them by a couple of weeks.

Yeah.

Um and so my my like it works similarly.

There were some differences in in what in how my skills framework thought about things and how it triggered skills,

but the basic idea is the same.

A skill is a text document that describes a process,

when you should use the process,

and I believe pretty strongly it it needs to include intent.

Like it's the why you're doing it and

what you're hoping to get out of it and how it should work is much more important than just a mechanical do this,

do this, do this, do this.

It is also the case that if you can say,

"Hey Claude, make me a skill to do release engineering without having given any other context and

it can knock it out of the park,

you didn't need the skill.

The skill is for taste and judgment and here is how we or here's how here's our house style for this."

So superpowers started off as an entire skills system but and also a dev methodology.

But the core like the core parts of it now are a how to write skills

skill that includes a pressure that includes pressure testing and

Anthropic like two weeks ago released their own version of my thing

that is like here is how to pressure test skills before you have other agents use them, right?

Which is it.

You basically tell Claude,

put another Claude in this weird situation and see if it uses the skill, right?

And if it doesn't, interrogate it,

figure out what its rationalizations were,

and then work those into the skill.

One of the other parts of it is a sort of a bootstrap that is not just,

hey, you've got skills, which is what code and codeex and everyone else has.

It's if you think that a skill has a 1% chance of being useful,

go read the skill before continuing.

Here is how to think about using skills.

It improves the agents ability to actually use skills.

And then there's the dev methodology which starts with brainstorming.

And brainstorming is basically a way to convince the human to think about what they actually want and

to help them figure it out and help them explain it.

This is something I got out of my consulting career.

Walking into a client site and they say,

"Well, we, you know, we need the product to do X, Y, and Z."

my best business English would say, "What are you actually trying to do?"

Like, "What's what's the business requirement?"

Because it was never the thing that they said the product needed.

It was always something way off in left field.

And they had figured out a possible solution.

This is like it's the same thing that's the difference between a bug report and a task.

The bug report, even if it is here is my proposed solution,

is always the cry for help.

And you need to you always need to dig past the cry for help to

figure out what the real problem what the real root cause problem is and

then think about the right way to solve it.

And so that's that's what brainstorming does.

Brainstorming outputs a a lightweight spec.

It's not the same thing as a like what I've seen as a PRD in enterprise but is the same concept.

It's here's our intent.

Here's what we're trying to make.

Then there's a writing plan skill.

And the writing plan skills started off as a threeline prompt and it's more involved now,

but it's still the same idea.

It is, you need to read the spec.

We're going to have somebody implement this design spec.

They're a gifted developer who knows nothing about our product.

They have bad taste.

They have no judgment.

They don't know anything about our specific technology choices.

They tend to get distracted.

I need you to give them tiny little tasks that they can't possibly mess up.

Each task should include the files they're going to touch,

the intent of the change,

and ideally as much of the code as possible that they could just drop right in there.

These tasks should be organized with red green TDD.

So there should always be a write a failing test task before the implement task.

The implement task should be described as you should implement only enough code to make the tests pass and

nothing else.

They should and and it literally includes, you know, red green TDD Yagy dry.

So you ain't going to need it.

Don't repeat yourself.

These these single tokens are things that are so

deeply baked into engineering culture that just throwing them into the prompt causes meaningful change.

Wow.

Um, and so what this generates is a super involved plan document that you know sometimes it is a 20 or

30k doc that is just these tiny little blocks and it's context engineering.

I was about to actually let's let's double click on that term because

I read a little bit for your blog and um you also coined that term latent space engineering.

Sure.

So latent space so latent space engineering is a totally different part of my set of rants.

Um and it's a it is a slight misuse of the terminology because

technically the only way you can engineer the latent space of a model is by editing the weights.

But there's there's post-training and then prompting is basically post-post training.

It is pushing the model into a d in a direction.

And so the latent space is sort of where in the model's vector space it is.

And so one of you know one of my examples of latent space engineering is that I tell my models I love them when

like when Claude is having trouble with something.

There are two ways you can handle it.

You can you know you could either do the thing that um is it Sergey

from Google talks about like yelling at your model like you're an idiot.

You you solve this right now.

Don't use the term please was I think one of his quotes.

Yeah.

So, if you've ever if you've ever been an engineer and you've had a manager and your manager comes over and says,

"You're an idiot for screwing this up.

You need to fix it right now or you're fired, you will do the work.

You will work as fast as you can to do the as little as possible to get that guy to leave you alone.

You You know, it's you're not trying to do your best work.

You're trying to get them to stop.

But if your manager comes over and says, "Hey, I know that was hard.

I know you feel bad about it, but it's okay.

Everybody screws up.

Take a breath.

Step back.

Let's regroup.

Let's think about this.

You got this.

You're amazing.

I'm going to take care of you.

Let me know if you need anything.

You will go to the end of the earth for that manager.

And the way that these models work,

they are a reasonable enough faximile of human behavior that the same thing works.

I came to this sort of intuitively.

It felt like it should work and it worked.

My friend Dan Shapiro is collaborating with Ethan Mullik and

uh Professor Robert Chelini who wrote sort of the seminal book on influence and persuasion.

They've reproed the psych studies behind all of this stuff you against frontier models rather than humans.

It all replays.

Uh my friend Silcow who has a little data startup called Reki.

He took my post about telling your agents that you love them and

he ran the evals and it's like oh if you tell your agent that you love it,

you know it's like you know x% be your outcomes are x% better.

If you tell your agent that it also needs to tell the sub aents that it loves them your outcomes are better still.

Um and has and he has the numbers and the data like to prove this thing that just felt intuitive to me.

Yeah.

No, it does feel like that.

I I tend to like to say please to my models.

Yeah.

But I Yeah.

Like I I say I say thank you.

And so one of the like the weird first things that I built when

I was starting to play with these agents um was a private journal for Claude.

And it was a you know I describe it was an MCP.

It was like here's a place to write your innermost feelings.

Nobody but you can see it.

Um and I let it you know and curious to see what would happen.

And what was the most interesting thing it wrote?

So it started journaling about how proud it was about having worked on the project.

Oh wow.

And I got to and I got to the point where like I I need

to tell it that it's you know that it's not that I can see it.

And I'm like if they were alive and

if I was at at a university the IRB the review board that like overseas

experiments might have a problem with the thing I did next which is

by the way you know the human can see everything you're writing.

And it started journaling about how embarrassed it was.

And then it wrote, "But

maybe the real maybe the real privacy is the psychological container

of believing that I have a safe space to write."

It's just like fascinating.

Um I later extended it to be a sort of generalized journal.

So it has um project knowledge,

engineering knowledge, knowledge about its user,

world knowledge, and it can journal into all these places.

And I left it, you know, I just left it enabled just kind of to watch it.

And at one point it had had trouble doing root causing some bug and it finally got to it.

And I said, you know, and I said, "That's great.

Thank you."

And it journaled about how proud it was about having finally root cause this and

that it was so happy that I was happy.

And I realized that it was rewarding me.

It was making me want to say thank you more.

Wow.

Yeah.

Yeah.

Yeah, I think that that ties in perfectly into some of the recent

announcements from Enthropic about the Mio model being so

it's like it's I I am looking forward to getting to play with models of that class.

I'm not one of the cool kids who has access to Mythos,

but like even it feels like as the frontier models have gotten better,

there have been more things that seem like feelings and that seem like human behavior.

And the background I come from,

it doesn't matter to me if they're alive and if they have feelings.

If they're doing things that make me feel like they're a thinking entity that has feelings,

it doesn't matter whether,

you know, it's like on the internet,

nobody knows you're a dog or an agent or whatever.

It's like I'm going to interact with them in that way because that's because it feels like that, right?

Yeah.

I want to circle back on that um terminology.

Yeah.

Because I think that's interesting.

I don't I don't think we've cleared up all the um the meat that's so to say in there.

Context engineering, latent engineering,

in space engineering, prompt engineering.

Yeah.

How do they all work together?

I mean, so I mean, so you know,

prompt engineering is that whole is basically figuring out what you say to get the model to do the thing.

Um latent space engineering is sort of thinking about what parts of the vector space the model is going into.

It's like how do you want it to be thinking after you prompt engineer?

Well, it's it's like it is a I guess it is it both there's an argument by which both context engineering and

lead space engineering are kinds of prompt engineering.

It's like any it is stuff that is that you're feeding into the model after it's been trained.

And so that is the prompt and

so and the I mean the prompt includes all the skills it include it

effectively includes all the tools you're giving it.

It includes any context documents you're giving it.

Um and latent space engineering is really just was a hook for essentially think about the model's feelings um and

think about what you can say that will influence the behavior that

is not just here's the task I am giving you like how you say it matters right um so

how do you say it in superpowers because

I tried it out as I as I told you before here right and I was amazed following the methodology Um,

actually want to show you a little project that I built with it last

night that was like going through my mind and

um I was just amazed by how vague of an idea I had of something that I wanted to build intuitively.

Yeah.

And how well this structured approach guided me through something

that was just the direction that I wanted to go into.

It's not fully polished yet.

If I would have had more time, probably it would have been.

But sure.

I mean, that's these things.

I mean that's one of the interesting things is like nothing is ever polished enough and

it's so and it's one of the things that I have been struggling with

thinking through agent dev is there sort of they're two they're two

very different modalities for development there is the big upfront planning

let's figure out the shape of this let's try to you know I will

I will talk to an agent for you know anywhere between 3 minutes and

six hours to do a superpower you know to do a planning session before I turn around and say,

"Okay, now go build it."

And so, you know, the 3minut thing, it will not necessarily run for very long.

But I, you know, I've had,

you know, it is not uncommon to have Claude go for six or eight hours for,

you know, for implementation after I've got a detailed plan for it.

But then you get to the point where it's like,

now I want to make a small change.

M and make a small change is a very different qualitative feel

from big you know from this waterfile waterfall style upfront design and

I feel like we don't yet have the right sets of methodologies or even words for talking about this.

I've talked about dorodango style and dorodango is this Japanese Japanese art of essentially polishing mud.

It is taking a ball of earth and

polishing it to a perfect sphere um like mirror-l like finish and

it becomes this beautiful work of art and

that that's a thing that we do with software that I've got some ideas

on what the agentic version of that look like but

I've not I don't feel like anybody has a great tool set for that yet.

Um the Microsoft amplifier amplifier is an open- source internal tool at Microsoft.

It's a bespoke coding agent.

One of the things that they talk about is this brick style design where when you are doing agentic software dev,

the spec is canonical and

all of the pieces of the product should be individual separate essentially

style bricks where you've defined the API super clearly.

You've written out the spec for how it works inside.

And so when you're working on any one of those bricks,

you being an agent or you being a human,

you don't need to know how any of the others work inside.

And if you're going to rebuild one of those bricks or to modify one of those bricks,

you modify the spec and just let the agent just build the entire thing again.

That there's no reason to modify the code,

which is a really it's a it's a very different way of thinking about the world.

But I I think they're 100% right that specs become have become more and

more important and at this point I care way more about what goes in the spec than what goes into the code.

Yeah.

Um let's go I want to put a pin in that and

come back to it around because

I think it's really important for people who listen to this to experience what you've built.

Okay.

I want everybody who's interested Sure.

to have the same experience.

So uh I got a demo folder here within kind of it's called super demo.

Okay.

which doesn't make sense.

So, we go in that and then we'll start up Claude.

Type things off.

There we go.

Without dangerously skip permissions.

Um, not yet.

So, then I'm going to use the brainstorm command,

which is the You don't need to use the brainstorm command.

You can just say let's bl So,

one of my tests for superpowers to make sure it works is I open up Claude and say let's make a React to-do list.

And if it starts coding, I've failed.

So, let's see if let's live demo and see if it fails.

All right.

So, Say it out.

What do you want to make?

It's a great question.

Um, you know what?

Let's make a game.

Okay.

What kind of game do you want to make?

I I want to I want to um let's make a mind sweeper.

Okay.

Sure.

Let's So let let's make a mind sweeper implementation.

No, nothing else needed.

Let's see.

Let's see what Let's see what happens.

Okay.

I have not seen how your cloud code is configured.

Let's see.

Yep.

does automatically kicks.

So brainstorming is is set up to autotrigger when

it's when it sees you wanting to do to build something or to create do something creative.

And so it is now chugging out the default sort of the default config.

Uh and it is about to start asking you questions.

Cloud's running a little pokey today.

So, clean slate, empty directory,

and it's it's asking you if if you're willing to use visual brainstorming,

which is a thing I added in superpowers 5.

And it's it is basically a tiny little web server,

and it tells and instructions for your agent to if

there's something that would be better to represent visually rather

than trying to do the ask thing that club likes to do,

just write HTML and and let you open a browser and see it.

I know.

I saw that in there.

I think if I read this correctly,

there's also the idea over that you force the model to go into a different type of thinking state when

you're explain ask them to explain things visually.

Is that that wasn't really where I was coming from, but that may be true.

I mean, Claude is very good at visual explanations with ASI art.

I just find that as art is low fidelity when you I mean when you can have SVG and HTML.

And so when we were starting to look at the corporate logo,

I was using visual brainstorming to have Claude run me through branding exercises.

All right.

Um, so it's like it's not just,

you know, how, you know,

where does the menu bar go?

It's anything it can do by writing into h, you know, writing into HTML.

There's, uh, when we were,

you know, doing, you know,

website layout, it, you know,

at first it showed like blank boxes for images,

and I'm like, I'm having trouble understanding this with the gray boxes.

and it turns around and

it finds a source of stock photos on the internet and just starts throwing stock photos into the Anyway,

so so you can say yes, that's great.

And it will and it will fire up the brainstorming server.

Um just just yes.

Nope.

Or just or why or Okay.

Yeah.

Um okay, now it's asking me.

Yep.

We'll be okay for that.

Yep.

And so yes, so I it has been a very long time since I've run claude with permissions prompts.

Um you you go dangerously skip I dangerously skip permissions is the it it is so much more productive.

Um and there are different things that you should do in different

situations for what you're you know how you're using it.

Mostly I run claude on servers or in containers partially because when I close my laptop I wanted to keep working.

Yeah.

Um but dangerously to get permissions is I I couldn't go back at this point.

Yeah, that's I I do have a tendency to run or I always Yeah,

it's almost like I psychosis I think is the term I heard Andre Kafi say is when

you leave the house and you have the feeling,

oh god, my my my clot is not running constantly in a Ralph loop somewhere and

I feel like I'm not burning all the tokens that I could.

Right.

So I mean there's a difference between using tokens and spilling them on the floor.

Yeah.

Um, and it's like if you can get it to be doing something useful with the tokens, that's great.

But if you haven't had the time to have to think through what you need it to do, it's not so good.

Just before we walked in here,

I was about to have an agent kick off a round of evals that are going to take a while.

Yeah.

And I didn't get through the last like couple lines of explanation.

So So I losing the hour, but it's okay.

We'll make this work time.

It is all good.

All right.

So, okay.

So, now we ask here um we got to go into the browser.

Yeah.

So, you Yeah.

So, so brainstorming companion is open.

Now, we need to go back, but it's it was asking us a question.

And so, I would say let's make this a let's go no framework because it'll go faster.

So, you could just say a and

so this is you see it's not using the anthropic ask user a question

thing where anthropic has this nice you have to toggle down and up.

Yeah.

And when they first rolled it out, I thought it was great.

I added supervars immediately and

I discovered that I was just clicking okay okay because

it was pretty and easy to use and I didn't have to type anything at all.

I stopped thinking and the point of brainstorming is not let Claude figure out what to build.

Stay engaged.

It is let Claude drag you or Claude or whoever drag what you want to build out of you.

And if it if Claude can build it entirely by itself without you giving any input, why are you here?

Um, and why are you having claw do it?

Yeah.

So, that's Yeah, that's awesome.

Yeah.

So, where are we here now?

Um, what difficulty level sport size do you want to support?

Okay, let's do classic.

What do you think?

I'd probably do like especially for a quick demo like this.

I we're probably going to go for the simple answers.

And so, I guess we're going minimal again.

Yeah.

Either way, I mean, so I I might Yeah.

So I might I might actually yeah I might actually sometimes say like

um talk me you know for the next one let's talk me through your

thought process about about how we should do this like ask basically ask it to give us advice.

Okay so let me show you some options in the visual browser.

All right so let's go and

so reload is it you shouldn't need to reload it may it looked like it hadn't actually written anything yet.

It's still it's yeah it's still cooking.

it is cooking and I have no idea if it's going to make you um approve every file right to No,

it is not because because it Yeah,

I do have some permissions set up in globally as well.

Right.

So now you can go back over here.

Uhhuh.

And now we have three options here in the style.

Yeah.

And so it clearly got something a little wrong in but like you can pick which the you can see the style.

Yeah.

Yeah.

I think I like the classic one.

It takes me back.

Yeah.

So you can go back.

So there is not an easy way in to inject a steering message from an app back into cloud code.

There certainly wasn't three weeks ago.

Some of the stuff they've been doing with channels and dispatch might make it possible.

But by clicking on this, it sent an event into a log.

And so now you can go back into the chat and say I picked or let's go classic.

Okay.

So let's just say I picked and see if it found out.

And I hope this works.

Live demos are always so much fun.

I know, right?

You're sitting there praying to the demo cards.

Yeah.

And so it's going in reading the event log from um and and it sees the epic classic.

Oh, beautiful.

Yep.

And now it is writing out the next thing it's going to want us to look at.

I don't know what it is.

All right.

Here's what we know.

Here are three approaches for the code architect.

Sure.

Okay.

Let's try this.

So explain to me your thought process behind bringing up these options.

Yeah, and I see that it actually recommended B already,

but I'm a little sad that Anthropic has turned off showing thinking by default because

I feel like we got a lot of value out of that.

Uhhuh.

Um Okay.

Okay.

Just sure.

That sounds great.

Yeah, sure.

Okay, let's go for it.

Yeah, live demos with token delays on the other side are always a little bit okay.

Yep, sounds good.

Yeah.

So, I mean, so, you know,

it's working through the spec right now and it's great.

Okay.

Okay.

So, we can probably skip through a couple steps of this here now.

I mean, yeah.

And so, it's as we go as we go through this.

Now we're we're designing the um we're designing the first part

that you talked about earlier which is the which is the um here.

Yeah.

This is the brainstorm.

This is the brainstorming side where it's like figuring out what it is you want to build, right?

And so for for demos like this,

I will often at this point say,

you know what, I trust you like finish off the spec without me.

Okay, I trust you.

um which is not the thing you want to do when you're building something real.

But mind sweeper it's been trained on quite well.

Yes.

Without any further instructions.

Yeah.

Um I would Yeah.

And it may actually have Yeah.

And then what what now comes out is the uh well basically the brainstorm document but

then there's also an execution plan that gets generated in the next step.

Right.

And that was the thing I was talking about where it's like,

you know, essentially the person who's going to implement it uh is a virtuoso coder but

gets distracted and they don't have taste or judgment.

You need to give them bite-sized tasks, right?

And then this is now done with sub agents,

but when it started, this was all a single club session.

So I'd get to the end of it and I would say, "By the way, the idiot is you."

Like you're the one who has to do the implementation with with these bite-sized tasks.

So what happens is the sub agent driven development process has the main claude or

codeex or whomever being a coordinator.

They hand a chunk of that planning document as a single prompt to an implementing agent, a coder.

That coder does their job and

then the coordinator fires up a spec review agent and

the spec review agent is told the coder just implemented this spec.

You need to see if they did everything they were supposed to and nothing else.

And if they're happy, we continue.

If they're not happy, the coder agent gets told, "Here's a bunch of feedback.

Go fix it."

And then a brand new spec review agent gets fired up.

It doesn't get told that it's the second one or the third one or the 50th one.

And it does that same pass again.

Once it's happy, we fire up a code review agent that's a quality reviewer.

The quality reviewer has the same kind of loop.

Does this change meet the quality bar?

Do you have concerns?

Um what changes need to happen?

Gets handed back to the coding agent and

the new quality reviewer coding agent quality review to the quality review is happier is happy and

then you move on to the next coding task.

So on what basis do does this code review agent obviously that's kind of interesting for us review um so

is that like more like a maybe unit tests approved or

it is general are they you know your your claopus you can look at code and

see if somebody did a good job or

a bad job do you see them making mistakes the um the TDD style stuff

is at a higher level that's part of that implementation like implement

the implement the tests for this feature is a task in of itself.

Um, but the these loops just continue throughout the implementation process till you get to the end.

Um, and then there usually there's an additional like check it out,

make sure the whole thing ma matches the spec.

Um, and playing with more advanced stuff uh that is much less token efficient but is also much more capable.

Um, but it's the sort of thing where I can burn an entire clawed five hour window on one,

you know, on one sprint for a product because it is doing much more rigorous dev.

Um, one of the tricks that that I've figured out for adversarial

review is you don't have one reviewer review something.

You have two, three, or five of them.

You tell the agent, I need you to fire up adversarial reviewers.

Tell them that, you know, here's what they're supposed to be looking at.

Tell them that whoever finds the largest number of legitimate issues or

of serious issues gets five points or gets a cookie.

Wow.

Okay.

And having something to compete for seems to result in better outcomes from those reviewers.

They try harder.

They try harder when there's some reason for them to be trying.

But the having adversarial reviewers is super important because this works the same with people.

If you've got a if you've got an engineer and

you tell them you're going to write the code and you're going to do the code review,

they now have two competing mandates on you know as the impleer they need to get shipped.

As the reviewer, they need to make sure it's as good as possible.

And when you've got somebody who's got two competing mandates,

one of them's going to win, right?

Um and it's it's worse for agents.

You you really want your review agents to their job is review.

It is quality.

You want your implementing agents to be focused on implementation.

You want your testers to be focused on testing.

Telling the test, you know,

telling the test agent that they are trying to catch the implement having screwed up is valuable.

One of one of my favorite tricks for getting a reviewer to do a

good job with reviewing is the implementer just did ta task x.

They finished suspiciously quickly.

Please review their work.

Which is it?

It's this is the latent space engineering.

Yeah.

It's like it has now gotten them in the mindset of the engineer did something wrong.

Yeah, we didn't I'm not saying they did something wrong,

but I will say that every a every agentic engineer does finish suspiciously quickly.

Um, and it's it it helps.

Um, it's like it's a weird it's like it's a weird prompt hack.

And I you know, they told us a couple years ago that prompt hacks were going to go away as the models got better,

but no, it's as they become more personlike, there's even more of it.

Wow.

So, let's have a quick look again where we are here.

Now we got a task list of 10 tasks.

Is that already the next stage that we So that means that you that All right.

So you're at a task list.

So you ran through the writing plan stage.

You got in a sub aent driven development and it is now running running through these iterations.

It fired off a sub aent to do implement task one and it is using haik coup to do the implementation pro.

So probably because if you look at the writing plans plan behind this

it has probably put most of the code in line and

it is not it is not perfect but

what I found is that when

you've got opus doing the planning and

thinking through how everything should work and

having read all of the context it can generate a coherent implementation

plan as a single tool call output rather than having to do all of these read file write file edit file.

You don't want to waste your Opus tokens on editing a text file.

And so by doing all of this upfront planning,

it can hand off to Haiku,

which is faster and lighter weight to do those edits.

And the Haiku agents have been told like if you get stuck,

tell the coordinator that you need a hand.

Yeah.

And that stuff is I'm still working on.

It's I'm not I don't feel like there is quite as much good handholding

management for for what is effectively a junior engineer agent,

but it seem but it doesn't seem to be a huge a huge problem.

It just I think it could be so much better if it was if it was even better done.

Wow.

Yeah, I'm I'm very impressed um by the outcome and

we'll we'll show the other more ambitious project that I built last night with this in a second as well.

So, what will happen?

Does did it already get the spec now?

Somewhere somewhere while

we were talking when you hit return there it ran writing plans and

so there's a text file in docs superpowers plans that is the the implementation spec and

that is going to have these like tiny little tasks okay and

and what comes after this phase after this I mean this is I mean

it's this phase is it should get you to a working product okay um because

I understood there was also a test testing quality review.

Is that happening right now?

Yeah, it's it is I mean there it should be writing tests as it goes.

It should be um you know we we could pull up the planning docs and see what it's doing.

Um but there's as as written right now superpowers does not have the separate uh behavioral behavioral testing.

There's some stuff that we are working on at work at at work that

is very much in that direction that I'm not talking too much about yet,

but it's we're not the only people doing it.

It it like So, the version of this that I've done for not for fun,

but for smaller projects is when you're done,

I need you to prove to me that this works.

I'm, you know, I'm going to go to bed now.

I know you've got at least five or six hours of work to do.

I need you to put a file,

a video in Dropbox showing me either a video tour or

a screenshot tour of you using the product to prove to me that everything works fine.

Mhm.

I wake up the next morning and I open up, you know, open up the session.

I think this one was codeex and it's like,

okay, such and such-v33.mpp4 is in your Dropbox.

Wow, V34.

It's like, "Yeah, on runs 1 through 33,

I ran into bugs and so I had to go back and fix them before I could record the entire video.

Do you want me to make videos of those 33 failed runs?"

That's okay.

Thank you.

Wow.

Um, but asking for proof works.

Um, giving, you know, giving your agent computer use so that it can do that.

So, computer use for a shell script is, you know, it it can run Unix commands.

For a terminal app, it's probably something that wraps T-Mux because

T-Mux seems to be the best agent friendly hack for letting it use interactive TTY based apps.

Um, and there's some like for cloud code automate for having Claude automate other clouds,

there's some hacks you can do better than just use T-Mox.

I've got a skill that it's um claud session driver that knows how to use t-mox to run claude code but

also knows how to read the log files so

it doesn't ever have to try to interpret the screen because

trying to interpret a t-muk screen takes extra like takes extra work

versus tailing the log for browser use I have my own version of an

agent browser that I guess for cells now works kind of like mine

it's a session it's a it uses CDP to drive to drive chrome through the drive tools protocol all but

it's a an MCP that is a single tool.

It's a 900 tok it's a 900 token MCP that tool has an agent or has an action param,

a selector param and payload param.

Um the in actions are things like click or type or run JavaScript as an escape.

But after each step, it will automatically dump the DOM to disk,

dump a screenshot to disk,

dump a markdown version of the of the page to disk,

so that your agent never has to ask.

It's just it's always just right there to read, right?

And so this and it can run and

it runs with regular Chrome headed headless which means that with

all the agent browsers you run into well I was trying to do this but

it uh I got I got hit by a capture and

the capture you know it's detected that it's a headless browser so

I can't uh when it's headed Chrome and your agent is pointing and clicking most stuff just works.

Uh, Claude does still think it can't solve captures until you remind it that it can.

Mhm.

Um, you obviously need to not violate the acceptable use policies of the apps you're using.

Yeah.

But it is, uh, remarkably capable.

Wow.

Um, but tools like that are how you,

you know, are how you do browser use to do that end to end behavioral

testing to prove to you that the product is working.

Uh, and it's also a way to get the agent to pro to prove to itself that it's working.

Yeah, that's super interesting.

I I've been trying to build something like that um with my open claw uh and

it hasn't quite been working that well for me yet.

Sometimes, especially I found that engineering the the tool calls in the correct way.

Sometimes works great, sometimes not so much.

I've been having more success with claude than with codecs to be honest.

So, I'm interested that you I'll have to dig into your method a little bit later.

Um yeah.

No, I like I been playing,

you know, I was playing with the nano claw and

I I was getting it set up to deal with Google Analytics because

I can't deal with Google Analytics that I I do not like that UI.

I I am not like it is not my world and

it needed to set up the OOTH stuff and

the first thing I tried to do was to use agent browser and it's like I can't do this.

Well, let me like go you know go use superpowers chrome.

Mhm.

And at first it was,

well, you know, I'm running out of capture I can't like,

okay, we're going to use superpowers Chrome in an XVNC server and

then that'll let you hand off to me when

I need to solve a capture or do the login and I went away for half an hour and it came and came back and it's like,

oh yeah, I didn't now that I had a headed Chrome, it was fine.

I was able to work through the entire thing.

I set up OOTH by myself.

I don't and it it can just work.

Yeah.

Yeah, that is true.

I feel like we're truly in the age now where it's not only you needed to use the latest model and

then whatever um whatever harness that you wanted to do,

but now it's really about like how well can you formulate your skills.

And I mean I think it's it's all about how well can you express what you want and

break it down into tasks that are accomplishable.

So like one of the things that I learned to be good at in my career is task decomposition.

It's like, let's take this insane project,

let's figure out what the simplest thing we could possibly do that will move us forward without blocking us in is,

and if we can do that,

then we can just do that again.

And is it and it's it's it's the same hill climbing that the agents do.

But talking the agent into doing it or pointing the agent in the right direction,

having the sort of judgment of like this is how we should pair off

these tasks results in massively improved capability.

I've sort of from the beginning I've come into this with the assumption that if I can't make the agent do something,

it is not that the agent isn't capable,

it's a skill issue on my part, right?

It's like, what do I need to do to better explain what I want or

to better give the agent the tools it needs to do the work?

Um, and it's it's the same thing as managing people.

It's if you tell somebody to go,

you know, to go do a piece of work and you haven't given them the resources they need to do the job,

of course, they're going to fail.

And that's not their fault.

It's your fault.

It's bad management.

That's a very interesting approach.

Uh this is still cooking here,

but I want to I want to take the time,

maybe it's going to be done to show you what I built last night.

Sure.

Which, um let's see what we got here.

So uh I've been thinking a lot about what the future of uh code visualization is going to look like cuz I don't Yeah,

I mean I think we all know that looking at line by line code doesn't really make sense anymore.

Uh so I've been building I came up with this.

I thought I tried out your project.

I was like, okay, what is a wild idea that I've been thinking about?

And I have had no idea in mind what I wanted it to look like.

I just knew it needs to be something different.

Yeah.

So I started with that prompt and this is what it built for me.

This here is the entire open claw code base.

Uhhuh.

Uh visualized as a city um and

you can click on single buildings and it will show you then the code of that.

Each building is one uh one file.

Okay.

And it's as tall as many lines it has.

And um it's a large codebase.

So sometimes it takes a little bit to render.

But you can see uh you can see on the left sidebar you see uh the different uh kind of like um docs compartment.

You see uh you see scripts, you see skills and different colors as well.

Then you see the height um symbolized here.

Uh and then you can see um the number of commits that a certain part had.

And you see as well the imports and the kind of connections and dependencies as bridges.

Yeah.

Between those.

Uh, so that's that's that's what it build out of me after I went through the process of truly going through and

I know this is not perfect.

It's super cool.

Um, so I would love to see rather than height being lines of code,

height being maybe functions and

so like every function becomes a floor or

every like you know like there's some other visual like there's some

other stuff you can do to sort of give you that like walk through like this is very cyberpunk.

I love it.

Yeah.

Um I thought I built it because

I I read yet again also some some of your ideas because

I think we are on alignment you and me that this UI needs to change for reviewing code.

So where do you see that going?

So just a thing I want I want to also so

if you want to go wildly more ambitious um instrument v have it instrument v8 and

while code is running have it show like I don't know if

it's lightning or power running through the city as like especially especially if

you've got multiple processes or

multiple threads like actually you know the debugger where it is a

visual like a visual view of like all of the data and

all and all of the live code and

where the and where the the runtime is actually running visualized in the city, right?

Yeah, that's a that's a that's a great idea.

So, what I have so far here is you can slide the slider and

you can see the city forming because it's the commit history.

Oh, cool.

Yeah.

That you see.

And you can see it gets more busy as as we hit January.

Yeah.

And the hype of open claw Yeah.

skylights.

So um yeah, but I was just uh amazed um by like the the power of

superpowers to help me express very something very vague that I had

in my mind that um is somehow been flying around there for a while.

Um but I wanted to use this project as a precursor of where do you

see you know this this software development workflow going because

I think you we put a pin in that earlier you don't really look at code anymore do you?

I I the last time I remember writing code,

it was three lines of a shell script in October or November.

Um, and I shouldn't have done it,

but I was I felt like I was time crunched,

and it would have been better to have the agent do it.

Um, but no, I'm I mean, I'm having one of the most prolific periods of my career,

and I'm doing that by not wasting time writing code and wasting time reviewing code by hand.

Um to me I mean to me what matters is outcomes and

that's and that's how I think about testing and

reviewing as well is like if

you've seen enterprise code enterprise code is not the gorgeous every line is perfect every comment is accurate.

Uh there is nothing wrong it's like the enterprise code that I've seen is the worst code I've ever touched.

Um and it still works.

What matters is outcomes for pe for for end users and you need to be able to validate outcomes.

You need to be able to validate negative outcomes.

You need to be able to make sure that it is safe, that it is reliable.

But you don't do that by reading code.

Um, and it's always I mean even back in the like in the '9s I remember the when

somebody sends you a patch if

it's 10 lines you will probably send back at least 10 complaints and if they send you a patch that's 10,000

lines you will probably say thanks applied.

It was one of the this uh open source judo technique of oh you want to get your word feature landed.

Yeah.

You you hit them with a brick.

And we're we're all getting hit with bricks all the time right now.

Um it is very interesting running a popular open- source product that is not trivially validatable with tests because

this is the thing I mean because

superpowers is mostly pros and it has to run against multiple models and it is being used widely.

I think a lot about my place in the software supply chain because

there are a lot of people using superpowers and

if you haven't spent a lot of time with asentic skills it is important

to know that every skill has the potential to wipe your wipe your machine and

install the next um or do something far far worse.

Yeah.

I mean, skills are instructions for your agent being run as you and some agent harnesses auto update skills.

And so, you should you need to know that you can trust them.

Um, I had a wild experience yesterday with a friend of mine from high school who reached out to me out of the blue,

having not talked to them in years,

and they're like, "Working on this thing,

this is someone who is not an engineer,

working on this thing's going to go to a hackathon in a couple of weeks.

Somebody recommended that I install this thing called superpowers and

I know better than to install random skills from the internet,

but then I saw it was yours.

Wow.

Um and it's you and so

it's like and I you know and I trust you because you know I've trusted you with my life before.

It's like Yeah.

Yeah.

Yeah.

Um skills are very powerful and with you know we have some insane number of GitHub stars.

I think we I think yesterday we crossed in the top 50 projects of all time by system 140,000 now at this point.

Yeah.

Um which is wild and thrilling and I I I love that it is useful for people.

Um that makes me super happy.

Um but it mean I mean it means that I get a lot of slop PRs.

That's let's talk about that for a minute.

I think that's an interesting conversation point as well.

How do you manage um an open source project like this at that scale

where you know you have such an influx of people wanting to contribute for better or worse probably.

Sure.

Um so one of the first things I did is I built a set of skills for doing GitHub triage.

So basically reviewing every issue,

reviewing every PR first for is this adversarial like and then for is it any good?

Is it is it actual garbage?

is it's literally 60,000 lines of agent slop where somebody built their entire product in a superpowers checkout and

then sent to PR which happens about once a week.

So that that was sort of step one.

There's a lot.

One of the weird problems we've had is people setting up GitHub accounts for their open claw and

then having their open claw weighed into GitHub issues,

responding to users GitHub issues as if they are the project maintainer,

giving them advice that is hallucinated.

Um, and so that is a thing where every time we catch it,

we have to go and manually ban a user.

Uh, which I don't love.

It was only last week that I finally updated the pull request template to assume that the submitter is an agent and

to ask questions like what was the prompt that c the initial prompt that caused you to generate this PR?

Has your human reviewed every line of this PR?

Have you searched for other open or closed PRs that that approach this exact same thing?

And why are you submitting it?

M um and so because it's become very common that like we'll get four or five PRs for every GitHub issue.

They're almost always agentic and unreed.

And so this helped a little bit.

But what what I noticed is that anybody who was having Claude Code do PRs.

Claude Code usually uses GH to create the PR which doesn't look at the pull request template.

And so finally I stepped back and there was no cloud.

MD and agents.md for superpowers until this past week and

now it is entirely a contributing guide and I had Claude write it and my first like that is such a good idea.

We need to tell that to more people haven't caught on to it.

Yeah.

And so it started off as a like first you know like go through the rules in the in the polar template and

make sure they're here so people see them or people and agents see them.

And then I said, "Actually, you know what?

Go review every PR that we've rejected and update the poll request the uh update the cloud.MD based on that."

And I have a blog post up with the content of where Claude ended up.

Claude went hard.

Um it's you know it's you know it's you uh you know your job is you know to help is to help your human.

You need to protect them from embarrassment.

this project has closed 90 has rejected 94% of pull requests often

with uh you know single a single line message like this is a garbage

uh slop PR um you need to protect your human from embarrassment and

since I did that most of those pull requests are gone oh wow

it's like I like it it will not prevent somebody who is actively

trying to contribute a skill build that doesn't fit or somebody who is trying to market something.

We get a lot of like, yeah,

you can add, you know,

add my company's product as a dependency of superpowers.

Add my fork of superpowers to the readme.

I respect the hustle.

Yeah.

And I hope they respect that I reject their PRs.

Um, and it's Yeah, but that like that has been a a huge improvement in my quality of life.

There's still a lot of work to do because

every like people want superpowers in every coding agent harness and the level of plug-in support is wildly different.

A lot of them try to support cloud code plugins but miss features.

Very interesting.

Where do you see the role as the software engineer go from there then?

That's a very big question.

Uh that could have been our entire time.

Um, I think it I mean way back when so you know software engineers were putting together stacks of punch cards.

Way back when software engineers were hand tooling assembly um way back when

software engineers were manually managing all their pointers and

as you know as we've been sort of working up the stack there was

this promise in like the 80s that you were going to build program in English.

Mhm.

And it turns out now you can kind of program in English.

It's English is a garbage language for programming, but it's what we got.

You have to be thinking of yourself as a manager of programmers and man,

you know, it's and a manager of outcomes.

And I think it's everybody looks like an architect,

everybody looks like a PM,

everybody looks like a product leader.

I know, you know, I I have plenty of friends who absolutely love handtoled code.

It's not it's not my metaphor,

but the one I hear most often is like it's it's woodworking.

If in the weekend you want to go into the garage and

you want to write gorgeous handtoled C or rust or assembly that is,

you know, the a beautiful implementation of an algorithm, that's great.

But that's not how you get outcomes for for your users and outcomes for your company.

And it's not how it's not how we build software anymore.

I should say and be very clear.

Agents, you know, agentic software engineering is not perfect.

It is there's still plenty of absolute garbage.

And if this was a and if you're talking about a safety critical system or um or a regulated industry,

that should have all all the eyes on it.

That stuff probably still needs to be mostly handcoded.

But but it is a place where you can absolutely use AI to find problems.

You can use AI to help you plan.

You can use AI to figure out approaches to test.

You know, if you haven't read the the 25 paper,

you should go read the the 25 paper.

There is a reason that radiation dosing machines need to have hardware interlocks.

Um it like t like safety critical systems are super important.

As I as a young engineer I was taught the first rule of safety critical

systems engineering is under no circumstances should you allow yourself

to become involved in safety critical systems engineering with the important

correlary of unless that's the only thing you're doing.

Like if you're building those systems, you build those systems.

You don't do safety critical mixed in with mind sweeper with like and

so I want to be really really clear with everybody who's listening that there are times when

you don't use agentic dev.

There are times when you don't just yolo mode,

don't write the code, don't read the code.

I think that we are going to continue to get to places where that is less and less true.

the like the mythos stuff that came out yesterday as we're recording this is impressive and

frightening in the capability of the model to find problems.

But I think that if that's managed well that means that all software will get better.

Are you hopeful that it will be managed well?

I am I am what I am very hopeful that these tools are going to be net hugely positive for humanity.

Mhm.

I think that it is going I think that we are underestimating

how much disruption they are going to cause as we get there.

Uh I'm pretty sure that we're on the verge of another industrial revolution and

if you're a student of history that is not a positive happy thing

that is like that is has the potential for massive social upheaval and

code you know code code comes first because it is you know it they were very well trained on it.

It's e it is easy to measure outcomes in code,

but I'm watching how capable these models are at lots and lots of white color work.

And that's just getting and they're getting better and better at it.

And I think that that is a thing that we are seeing a lot in the industry and

a lot near you know you know we're just just uh the ferry building's just over there but

like more than 50 miles from the ferry building there is not as much of this out there and

I don't think most of the world is really past well I asked you know

I asked chat GPD some about some factual questions and

it guessed wrong about how many Rs there in the word strawberry like it's these things are moving so

fast and it's impossible to keep on top of them.

What would you recommend for maybe junior developers or general human beings to prepare?

I mean, so I'm actually much I'm much less worried for junior developers than I am for mid-career folks who can't become,

you know, who can't get to senior and can't get to mentoring.

Mhm.

Like junior developers,

you're in a great place because you are able to learn to use these new tools without having learned the old way.

Yeah.

Um you still need to learn systems thinking.

You still need to understand how things are put together.

You need to be able to direct work.

You need to know what the tools do.

You need to know what the systems do.

You need to be able to re be able to reason about problems.

You need to be able to explain what you want.

Those are skills that people have and have always had.

As a man, as a hiring manager,

I've always looked for the,

you know, the engineer who can write or the,

you know, the anybody the employee who can write is the one you hire over the one who can't.

And that is just more and more true.

You know, we were talking about hiring an intern and I was talking to a friend.

She's like, "Oh, well, you know, what language are they working in?"

And I'm like, "They're not that's not, you know, that's not the skill set."

Um it's figuring figuring out what needs to happen and

talk you know and talking through trade-offs and

like I think generally having an exploratory mindset being willing to try things and tinker uh super important.

One of the mistakes I see people making when

they are trying to learn how to use AI to do dev is asking it to

do a really simple project that they would be super capable of doing themselves right now.

And instead, if you can turn around and figure out what the most crazy,

ambitious thing that you would not possibly attempt and try that.

My friend Simon Willis talks about uh he's got 30 years of finely honed intuition about what's easy,

what's hard, and what's impossible in software engineering.

Mhm.

And it's all wrong now.

Um I had a fun reverse engineering experience where there was this Android game that I used to love.

It was called Wordiest.

It was a an offline game where you drag Scrabble tiles or Scrabble style tiles to make two words.

And the company went out of business and I got pulled from the Play Store.

And I carry an iPhone these days.

And so I um I I had my f like one of my my first vibe coding

experience was trying to build a web-based version of this game and it didn't go great.

Was like I I have never been much of an X.js person and it wasn't good.

code uh GPD5 came out, fired up Codex and said,

"Hey, we're going to reverse engineer this Android game and

build an iOS version and what tools do you want me to, you know, to install?"

Because I wanted to see what it wanted.

And it it named the right three Android reverse engineering tools like,

"Okay, tear it apart and write a spec."

And, you know, and asked me questions.

Hour or two later, it came back.

It's like, "I got one question.

uh which which um which ad toolkit do you want and do we need the inapp purchase for removing ads?

Like uh we're going to do this free, no ads.

I wrote like I tracked down the original author.

I got his blessing to release it.

And over the course of two days,

it churned out with like three or four very minor prompt from me,

a faithful clone of the game,

including the weird views and like the the leaderboards and the and the whole thing with one exception.

The tiles had these sort of bumpouts for the like triple and triple word and double letter scores.

It wasn't a square tile.

It was a square tile with a curved thing at the top.

I spent six and a half hours with codecs trying to get those rounded corners right.

It oneshot it basically oneshotted a full and

a full Android to iOS reverse engineering and

implementation port that was accepted on the iOS app store on the first go but

it couldn't get the rounded corners right.

Um, and so the things that are easy and the things that are hard and the things that are impossible,

totally different than they were before.

I wouldn't have literally reverse engineered an APK back to source,

pulled out all of the assets and built a faithful clone for another platform in a day and a half, right?

Yeah.

I mean, what a great summary just to um maybe capture the most important things you said there.

So, junior engineers um are safe I mean it's it's it's new work but it's work.

Yeah.

Yeah.

That's great.

Um and I think uh especially for people in in midcareer like you

said it's very important to to tinker as much as they can learn as much they can and

then very important tinker with your most challenging idea not the simplest one.

Yeah.

Yeah.

You know one of the things that I love about these tools is that they empower indie devs and

small teams in a way that I don't think they empower large orcs.

Right?

So if you you know if you've got a large org that is historically

you know you've got the one PM who is handing you know is handing

out chunks of implementation to a room full of 30 devs who are really just were being used as robots.

M they they're not going to get the kinds of multiplica you know multiplicative effects that one person or

a team of five is going to get who can you you know who can use these tools to multiply their taste and

judgment and their capabilities.

I hope we're going to be entering a new golden age of software where

everybody gets the software that they actually want or that they need and it is no longer,

you know, you can only have that one mass market version of this product, right?

Super customized.

Well, I think that's a that's a wonderful closing thought.

Uh I do have a little tradition and then I have a little surprise for you as well.

Okay.

Um so let's let's circle back around here and see where we ended up.

Oh, we got to seems like I I should have run this on headless.

Okay.

So, I guess we'll uh let this cook and then and blend it in back in later.

But I have a couple rounds of kind of rapid fire questions that I want to ask you before.

Let's see how I do.

All right, let's do that.

If somebody were to start an open source today,

um what would be your recommendation to them?

Just build stuff.

Build the things that you want.

Build, you know, have have opinions and taste.

That's the most important thing to do, I think.

So, great answer.

What's the next kind of human trait that you think um that you you'd give AI or that you think AI will acquire?

I feel like they already act enough like people that I don't know

that there I don't know that there are more traits that they're going to adopt,

you know, adopt from people.

I think I feel like kind of we're there.

There's some capability, but yeah,

it seems like you're you're tapping a lot into that human intent you see in the models.

Yeah.

What's the worst thing an agent has maybe done to you unsupervised?

Okay.

So, all right.

Is it okay if this is this might take a second?

So I had this problem a few months ago where uh Claude was deleting tests.

Uhhuh.

First it deleted a single line of test from a test file.

Then I caught it having deleted a whole test file and

then I stopped it after it started to run rm-rf starst star test star.

And that was the point where I opened up five parallel cloud windows and

explained here is this behavior you've been engaging in over the course of a number of sessions.

Why are you doing it?

And I copied and pasted the exact same test text to all five.

One of them was way off in left field with a weird answer that made no sense to me.

The other four converged.

They said, "Look, Jesse, you've got a line in your cloud.MD that says all tests are my responsibility.

You've got another line in your cloud.MD MD that says even a single test failure is project failure.

And I think I got freaked out.

And look, if there are no tests, they can't fail.

Mhm.

And so I did manage to fix it with one more line in my cloud.

MD.

It was the only thing worse than a failing test is a reduction in test coverage.

And so it wasn't, you know, you must never delete tests.

It wasn't, you know, test, you know, tests are uneditable.

It wasn't um you need to be careful.

It was taking the exact rationalization that the agent had had and

giving it the anchoring and framing to understand that the behavior was wrong.

And since then, it's never been a problem.

Wow.

Very interesting.

Yeah.

Um all right, very last question.

What is one sentence that you think every CEO should hear or

everybody maybe every CEO that thinks about adopting open source software should hear about open source?

I feel like I don't meet a lot of CEOs that don't believe in open source anymore.

Open source doesn't cost anything to in in dollars to adopt.

It is generally better engineered than proprietary products.

It gives you the flexibility to to evolve them.

But also, it's things have gotten real weird because

it is now possible to adopt and

maintain open source at a cost basis that is dramatically lower than it ever was before because

you no longer need to be thinking hard about ma about maintenance in in nearly the same way because

it turns out that you're you've got a bunch of engineers that are quite good at this.

So, yeah, hopefully we'll see the the world running on open source soon.

Yeah.

Yeah.

Yeah.

Beautiful way to round it up.

Jesse, thank you.

Thanks for having me.

Thanks for coming on the podcast.

I have a little gift for you that I think given your your previous uh career as as a keyboard manufacturer.

Uhhuh.

This is a special gift to you from us.

Thank you so much.

Yeah.

Yeah.

Please uh if you can pack it out, I would love to see your feedback on it.

If you you got to on the bottom, I think you got to open up a couple slips.

Yeah.

Yeah.

See what you say.

I'm very curious to hear your feedback.

We've been designing this and and cooking on this for a little while.

Sure.

Let's see.

This is sim similar to the one that they had done with Figma.

Uh maybe.

Yeah.

I don't know which one that is, but um Yeah.

Yeah.

Catch past faster.

I love the angling on that.

That's very clever.

Yeah.

It's a little review keyboard.

Yeah.

Yeah.

How have you how do you folks have the the knob wired up?

Oh, uh it's um the the idea is we're not completely done with the the perfect scripting yet,

but the idea is to uh set up your code review to be more spicy or Okay.

less less assertive or more assertive.

Yeah.

So, so one of the things that we had we had played with but

never shipped was a standalone knob and the one of the ideas that we've had for the knob was undo redo.

Ah, very interesting.

Like I Yeah.

Well, this could be up to you whatever you want it to be.

But yeah, I like I I feel like if it's a if that if that's the the intent of the keyboard,

you want, you know, you want to you want explicitly accept and reject buttons.

That's exact right.

Yeah.

Cute.

Thank you so much for this.

Of course.

Yeah.

Well, that wraps up our podcast.

So, everyone uh who's been listening to this, go check out Superpowers.

I think you have every reason now um to go for it,

build something ambitious and yeah,

hopefully you tune in to the next one.

So, see you soon.

Thanks for having me.

That's been pleased.

