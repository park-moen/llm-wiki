# From Backend Engineer to Head of Mobile (Lessons from Uber)

> Source: User-provided transcript file: pjRdtYRmqwE_Google.md
> Collected: 2026-08-16
> Published: Unknown

원문은 사용자가 제공한 자동 자막 추출 상태 그대로 보존한다. 고유명사 오인식과 문장 분할 오류를 임의로 교정하지 않았다.

## Original extracted content

# From Backend Engineer to Head of Mobile (Lessons from Uber)

The world is is changing every month.

It's scary to be a graduate engineer nowadays.

AI is not going to take your job.

The person who knows how to use AI better than you are, they will.

Pick your battles. Disagree and commit.

I love to build things and it was tough to realize that.

You put so, so much effort into building something that people don't need.

How do you build a career, specifically a software

engineer, developing apps for mobile?

That's what we discuss today.

And we talk about AI assisted code tooling.

How to leverage that to make yourself more productive and faster than ever before.

Joining me today, he's head of mobile is my good friend Pasha.

And previous guests and friends of mine have told me

great things about Pasha, and I can see exactly why.

So enjoy.

Beyond Coding.

I don't talk to many mobile engineers, and the only experience I have

has been in React Native specifically.

But I'm very curious in your work experience

or more on the productivity side nowadays, on a day to day,

what do you use in our tooling that makes you very productive

in what you do or what is effective in the end?

So it's mostly code.

I've got a console or multiple consoles

with with cloud open and

basically I'm trying to use it as a

junior slash medium engineer, fellow engineer

who can help me bounce off ideas and who can help me with code reviews.

For example, review my code.

In in the first place, and obviously to write some code as well.

And, so as long as you have a solid foundation set up in terms of,

MD files that help

to give context to, to, to the AI,

I think it can be very helpful in, in this regards.

Definitely.

So I recently joined a new company and

definitely I was super helpful

when it comes to learning new code base.

Right. You can ask anything.

What kind of features do I have in this code?

What are the entry points to these features?

Least amount and point me to the files.

And then I go there.

I look, look at the code and I understand

wholesale orchestrated and how it all works.

You know this with multiple terminals as well.

Yeah. How do you manage that?

So it's tmux, basically a,

split of four, tmux windows.

Yeah.

And that's something also I started to use very recently,

before I didn't use windows splits at all or terminal splits rather,

but with cloud, it's actually

very, very useful because in one, window, in one pane, I guess it's called

you have, a feature planner,

an AI who is writing a spec in another pane.

You have, an AI who is following the spec and is implementing the thing

in third one, you have an AI who would review things,

and you don't pollute the context of each AI.

So you kind of have to remember

which pane is responsible for what.

But then in the end of the day, you,

you get, pretty fast workflow with that.

Interesting.

When you said multiple windows, my assumption

was that each window would work on another feature in isolation.

But this is not what you're doing.

You're actually doing one feature,

and then you have different kind of purposes within a pane.

Yeah. Yeah.

So I have got

multiple repos of the same project set up.

So if, pull request is in review, if I consider feature done ish,

then I can move on to the next feature in another, another repo.

So similar to what you described.

Interesting.

I haven't done that yet because I have been trying this thing out.

I went from product management back to more hands on software engineering,

and this is one of the first things

that I actually struggled with was, okay, one feature is done, I'm going.

And I did definitely have some context solution

because then I go to another feature within the same conversation, even

without clear my context, and it goes, oh, all the changes that we had are gone.

We need to add them again.

And I was like, okay, so this is this is definitely a user error here basically.

And then I do hear people with multiple terminals

and indeed working on multiple features, but I'm like,

how do you fix or how do you go in specific feature

branches and you've solved that with actually having multiple repositories?

Yeah. Yeah.

There are work trees.

Okay. Git can can do work trees.

One thing that I didn't try that yet

I did try work trees back maybe ten years ago.

And one thing that, kind of put me off of using work trees

is that you cannot have two different work trees checking out the same branch.

I'm not sure if this is something that is fixed.

So you kind of have to work trees checking out develop basically.

And that's something that I usually do.

I check out, develop I, pull latest changes.

I work in, develop. I don't push.

And then I have, a command that would just check out a new branch, and

I'll put the

command to check the new branch and, and, finish it.

But for that, I need to stay in development.

That's kind of how I operate, how I used to it.

So for me, it's easier to have multiple repositories.

Gotcha.

Is there more you can share that makes you effective with regards

to AI assisted go tooling?

So I find that

our case is pretty successful in terms of AI.

Because I know that people,

sometimes they struggle.

And there was a study that I don't know how many 95% of organizations adopting.

I don't see a efficiency increase.

And, for me personally, I see efficiency increase.

But what I do is I actually use them

as a, as a junior engineer, not letting them do everything for me.

But rather review their code.

Their AI is code, and we,

review specs, iterate on specs.

I read what they write, basically, and.

Yeah.

And that is, I mean, I agree, it's something you are right now.

It's not something that I want to let go yet fully trust in autonomous agent.

Maybe it's also the project that I'm in, but I feel like it makes sense.

Right. The code that it spits out,

I feel like I'm reading way more code than ever before.

And I've also noticed that then me reading code specifically,

I get better at reading and understanding code

than actually writing, and even though I'm not writing as much,

the skill of really good has always been there.

It's just more on the forefront now and actually enjoy doing that as well.

Not just reviewing my own code, but reviewing other people's code.

And AI is just another artifact of that.

That's why it's so good. Absolutely. Yeah.

And then I that assists you in reviews like copilot the AI.

Right? Yeah.

It's a great way of pointing out places in code

that you have to pay more attention to and not necessarily agree

with everything that it would say, but it would point out to a place

where you probably would skim through and just say, it's fine,

but then you would stop and read the comment and start thinking,

if that's actually a legitimate issue or could be closed.

So yeah, you mentioned markdown files as well.

Yeah. Well, what do you do?

What do you put in the markdown files within your project.

We've got multiple markdown files.

There is a main markdown cloud.

And then

we have an architecture overview of a separate markdown

analytics overview issue reporting

and include that, do we reference to these files.

We say whenever you implement this your reporting and read instructions

from this file, and follow it.

Yeah.

Would you recommend whenever you do a it's just the go to a link

to set up kind of a similar structure in markdown.

I would definitely do that in the future.

Yeah. I like it.

It works for me.

I can definitely see how this could be,

in its initial investment that you have to do in,

But it pays off for me.

It pays off for us as a team. It pays off.

What do you mean with that initial investment, as in, to set that up?

Yes. Okay.

Yeah, you definitely need to guide.

You can write them yourself.

You can generate them with cloud as well.

But it can hallucinate.

It can put in things that do not exist in the code base.

You have to be not fluent in the code base, but it's

confident enough to review these files and

otherwise if it

puts some architecture patterns that you don't actually follow

in architecture, that then all of a sudden you start getting very weird output.

So these markdown files, are they instructions or are they like

also something that it continuously adjusts and improves

continue continuously adjusts and improves.

Oh interesting.

And it's also a good kind of hygiene habit.

So whenever you work on a feature,

if you do this with cloud or like whatever

AI tool, you mentioned if there is any relevant changes

that you have to do, in these markdown files, please do.

Yeah, interesting.

Especially with your experience coming as a mobile engineer,

you were first very fluent with regards to the iOS ecosystem

and now you're doing also, I'm assuming Android ecosystem.

Like how is that knowledge kind of crossed over?

Which knowledge?

Sorry, the knowledge of the iOS ecosystem

and now the implementation on Android side.

So I know

as a mobile engineer and I did work with Kotlin before.

Yeah.

I wrote in Kotlin for half a year.

So it's not that I'm coming with zero knowledge,

but definitely not a lot of knowledge that would,

that that proficient professional

Android engineers would, would have.

So with cloud, it's

a lot easier for me because I know how a feature

should be done and integrated from the high level perspective,

but I actually don't know SDK

and the Android way of doing things right.

Some of them are very alien to me.

Some patterns are very alien to me.

And then I lean on to code

to actually write this thing for me.

I will get to the bottom of it and understand what it did.

But it helps me to,

you know, get this over this initial hill

of actually implementing the thing that that would work.

Yeah, it does the knowledge transfer like of the mobile ecosystem

in one, let's say iOS versus Android three degree.

I guess you can

write, you can follow some patterns even on the back end.

Right. It's general programing.

Right.

But I guess with UI and compose,

which are I as an Android UI libraries,

they follow somewhat similar principles.

So it kind of,

transfers, but not to 100%, you know, for people

that are new in mobile engineering at all, how would they get up

and running fast with the tooling that's available nowadays?

Because I feel like because you have strong

fundamentals, you can like, you know, the patterns in one ecosystem

and then you can see what is alien to you, but you have that frame of reference.

And most people that are new, they will not have that.

It is a very interesting question.

I think that nowadays

it's more important than ever to

read and understand the fundamentals first.

Right?

Not probably not leaning on AI to implement everything for you.

I know how tempting it is to vibe code your first application so

that's what I want to do.

Yeah, but then you're missing out on on learning the actual thing.

And maybe in two years you wouldn't need to learn this, I don't know.

But my take is that you

you have to set up fundamentals yourself.

Yeah.

When you onboard new people, whether it is either in the company or in the team,

do you also check for those fundamentals specifically?

During interviews?

Yeah.

I mean, so I didn't do many interviews lately.

But I

would definitely check for those fundamentals. Yes.

And has the interview process evolved with AI tooling

that's now available, or how do you assess someone?

So in my view, classic interviews, they are kind of they should be gone.

Okay.

I can definitely see how for bigger companies that would

still be a way of understanding

if the person is worthy of of getting into the company.

But for smaller companies,

whenever a person comes to an interview, they would probably

or engineering interview or other, they would probably do

some lead coding for, I don't know, a month.

Yeah,

they would pass an interview and then they would completely forget everything.

They would do it coding because leetcode, because

all they would do is change colors on the button

to drive this revenue growth 0.1%.

Right? Yeah. That sounds like booking. Yeah.

So it is kind of pointless.

And for smaller startups, I hear more and more stories

when they basically do some initial filtering

and then they offer, a trial week for the, for the person to work with them

to actually understand whether, this person is good to work with.

Right.

Because that's the most important part, especially for smaller

teams, for smaller startups, that you still have the cohesion,

that that the other person has the work ethics.

Not that they can I don't know.

They know what they're threaded binary trees. But,

yeah, I think that's the future of the thing

because of of interviews, because,

technical part specifically. Right.

You would still probably want to have some, filter

in terms of like behavioral interviews and whatnot about, for the technical part,

with AI, it you don't need that much.

Room

resourcing, lengthy length list, sort of constant memory.

Now you don't have to have a top of mind.

So our our interview process and this is what I did six,

seven years ago was like a take home assessment for eight hours.

Because indeed, like you mentioned, company doesn't quite believe

that is leetcode and also in consultancy and software engineering.

It doesn't translate to what you do on a day to day.

So the project is more in line with what you would do.

And it's actually a project.

And you can choose

if you focus on front and back, and if you actually deploy your things,

if you make it up and running, you get a lot of room to experiment.

And then I go to and it came around and we saw a lot of people

within the funnel use that.

And a lot of conversation was, okay, are we going to allow that?

Or should we be explicit and say, don't use this?

And then we made a conscious decision.

People can use whatever tooling they want, but then be transparent about it.

Actually say that you've done this

and we'll have a different type of conversation,

because then we're going to talk about how

well you understood what it was generated. Right.

And if you can actually read it, or if you maintained it,

or how you set up your project to be able to execute on this

and what you think of the efficiencies or the trade offs there specifically.

So within the same interview process, it's still the same assessment.

We've kind of tailor made it towards people that do use AI assisted go tooling,

and we've now even gotten to the point where we expect it.

And when someone does it, then we feel like

maybe you should, because that's kind of where the industry's going.

Quite interesting.

And then, looking at your code, maybe you should have stuff,

but not not from that. That's right.

But yeah, I mean, in a general sense, yeah, I definitely know like your take on

or what you've seen other companies do in doing like a trial week.

I don't know how possible that is with regards to like Dutch law and stuff.

But in essence, if we disregard all of that,

I would love to work with someone on a day to day for a week.

And that form kind of the criteria of do I want to work with this person or not?

Because then we've already done it.

Yeah.

And I feel like the people that would go through a process like that,

maybe it's

hard to put myself in the shoes because I do go from assessment

or from project to project more often, but I would feel more comfortable

if I have a longer time to actually show what I'm worth, rather than a one hour

kind of system design or a conversation or leetcode thing.

Absolutely. It's a bit more lenient.

Yeah.

And nowadays a lot of companies are opting for, remote interviews.

Yeah, not even onsite where you can

casually talk to a person where you can have lunch with them.

And that's how how I was, how my interview was a tuber.

We, we had lunch together with the team I was interviewing for,

and I guess that's a little bit of,

Like, you have to spend time setting these things up, right?

But then you are that little bit more confident that the person is not,

let's call them, like, a bad person.

Still a week, with the person in the working context

will give you a lot more insight than any kind of behavioral interview can.

Yeah.

What specifically do you look towards, or do you look for

in the people that you work with or collaborate with on a day to day basis?

Be nice, be nice.

Yeah, that's that's if you're nice to other people

that can, open doors that you, you wouldn't have open otherwise.

Right.

Other people can go extra mile for you if you ask them something,

if you are nice to them.

Yeah, right.

And I think I read this article that Google made a study,

that it's the ultimate

key to, to working together.

And if you're nice to other people,

you're successful.

If it seems successful, maybe it's the way I grew up, but for me, that's.

Have you been in an environment where people are not nice to each other?

Because I do think that, I mean, we're knowledge workers,

and if you have in-depth knowledge and expertise within a certain topic,

you might be arrogant and that might kind of take away

from your kindness towards others.

That's the thing that I've seen.

I can definitely relate to that, that

the people who have strong opinions, and sometimes I do have strong opinions.

It's not that I'm, you know, like, fleshy.

You go the way that I,

But you definitely need to know

where your opinion matters, right?

And then whenever, if, if you're making an argument

out of every single point, then

your argument kind of diminishes, right?

Your your opinion.

You will struggle to make a point,

if you know what I mean, if that makes sense. That

everybody would see you as a person who is always against

some things, who always wants to things be their way.

Yeah.

You cannot differentiate between what is important.

Exactly. Yeah, exactly.

But then whenever you are okay with, you know,

one of the things that I love to follow is, disagree and commit.

And if I'm in the minority, if I see that

other people, smart people, they they have different opinions.

I'm fine saying, okay, I'm,

I'm committing to following your path.

Yeah, I'm fine with that.

But then whenever I actually feel

something strongly about it, then I will try to convince.

And other people would

listen because they're not used to me doing this kind of thing.

Right.

So I think that's that's also important to pick your battles.

Yeah.

I feel like I used to be very idealistic and like,

you kind of make a point out of a lot of things.

And now I'm trying to be more. Indeed.

When does it matter?

Is this going to be a key differentiating factor?

And I understand, disagree and commit.

I feel like if you read about it you will understand disagreeing, commit.

But understanding and behaving according to that is very different.

I've also seen people

say yes, I agree with you, but and then the but completely like

just takes it out of the water for me, that kind of undermines it completely.

And especially within a team where you have a lot of people with

very great skills.

Let's let's start with that.

But also because of the very interesting

opinions, discussions can just go and snowball.

And you don't need a specific either person or team

mindset that just says, okay, these discussions are no longer valuable.

We need to cut it, and we need to make a decision,

and we need to go with that decision until we find otherwise.

Yeah, basically. Which is also fine.

Yeah. And no. Absolutely.

And another thing that, that I learned

in my

previous jobs is that this state of analysis paralysis

where different people,

smart people in the same room, they cannot find the agreement.

So, a more senior person has to stand up and say, okay, we're doing this.

I see that this is not going anywhere, right?

We are going to discuss this today and the same with the same person.

We are going to discuss this tomorrow.

And in five days nothing's going to change.

So we just follow this path and we disagree and commit.

Gotcha.

You mentioned kindness.

And one of the things that you're looking for, being nice to each other.

What other things are you looking for?

I guess that, well,

obviously like being smart and being proficient in what you're doing.

On the other hand, I understand that

some things,

you know, at Uber, my manager used to say, I think that

that stuck with me probably forever is we are hiring people for their strengths,

not lack of weaknesses and

trying to identify those strengths.

During interview or the trial week or whatever you're following.

I guess that's one of the

one of the key, traits of of any interior.

And these strengths, they can be anything.

Right?

And they actually have to be different,

because you you want to have a variety of people

different people in your team.

If you are hiring

ten little copies of yourself, you are only going that far.

Yeah.

As I think it's Steve Jobs who who used to say, if you want to go fast, go along.

If you want to go far, go together.

But together,

if you're hiring

copies of yourself, you're not, you know, you're slow.

Yeah, in my books.

I mean, me as a little kid, but that made complete sense to me,

because if I could copy myself, I would go and be more effective.

And nowadays, as an adult, I'm like, yeah.

Then you don't accommodate for any of the downsides that you have.

Yeah.

How well aware are you of the strengths that you have,

specifically you as an engineer?

Because if I were to if someone were to ask me, what are your strengths,

I would definitely have to think about it for a bit longer.

Before I can say this is really what I'm good at.

Yeah, I would definitely need to.

Yeah, I think,

but I've talked to a lot of people and specifically about you,

and they do say you're a great engineer, which I think is quite, quite cool.

And we need to think,

the problem where do you see

the mobile and specifically the app industry going?

The has it changed with regards to AI and some of the apps that are out there?

I feel like a lot of people are creating little startups, and then their artifact

is an app that launches on the App Store more so nowadays than previously.

But if I were to start

my career, would a mobile engineer still be a good career choice?

From your perspective?

I think engineering in general

is probably not the best career choice nowadays, and

given the amount of of junior roles that are open.

Yeah.

And I have no idea how it's going to go, but looking at how it is

now, maybe in ten years, engineering is going to be very sparse.

Yeah. As a field,

but, Yeah, it's it's

such a difficult question with the.

With the space, with the pace

that the world is, is changing every month.

Yeah.

I would refrain from giving any recommendations.

That's for that regard. Yeah.

And I definitely feel very, Well, not bad, but,

like, it's scary to be, graduate engineer nowadays, I think.

Yeah.

I didn't expect you to go on the side of there.

Might not be like, I understand there's not as many job opportunities

as, let's say, Covid. What?

I think that might be the peak with regards to job opportunity that we had.

And now definitely

when you talk about peaks and dips, we're definitely under the lowest side,

I think with regards to job opportunity and maybe I'm hopeful.

Maybe I'm naive,

but I do think skills will evolve and there might be more emerging roles,

even though a lot of the things that we do day to day, they are getting automated.

Absolutely.

And I think I loved,

an I take I don't remember who said that, but I, the, the, the,

the saying goes that AI is not going to take your job.

The person who knows how to use AI better than you are, they will.

Yeah. So,

definitely the industry is evolving towards, more AI usage.

And if you're not using AI now, probably you're missing out on something

that in the future could be pivotal.

For, for your career. But.

Well, you said that there are a lot less job opportunities nowadays,

but there was actually a study, I think, from Harvard, that

there are a lot more.

Well, not a lot, but there is a rise of senior

plus opportunities, jobs, but

a huge dip in, in general junior roles.

So it's much easier to get hired as a senior plus engineer.

Yeah. Yeah. Interesting

talk about career perspective specifically.

I know you have an interesting story and how you got into mobile.

Let's let's start there.

How did you get into mobile in the first place?

I think it's,

for, for everybody.

They would not be able

to pinpoint the moment in life, that, that.

Well, not for everybody, but most people would not be able to,

pinpoint the moment in life.

For me, it was very clear.

My manager back in 2008,

or not, 20, 2006, I think it was they came into the room

and they said our company got, got a contract

for, Mac OS application.

And we didn't have Mac OS engineer, so they had,

you know, Mac mini in their hand,

and they put it on my desk and said, you are going to be our Mac OS engineer.

Yeah. That's it, it's you.

It's, Yeah, I was surprised to say the least.

But,

yeah, I, I

learned, I learned, with, I made a lot of mistakes

along the way, and obviously there were no,

you know, tooling that is similar to what we have now.

And documentation was a lot, a lot more sparse.

I had to learn, Objective-C on developer.apple.com,

and it wasn't great, to say the least.

So I made my fair share of mistakes,

and some of the mistakes were, really bad for the company that I worked for.

Yeah.

They they, lost the contract that they came,

that that got me into into Mac OS,

because of the mistakes I made, but also

kind of when I was iPhone SDK,

it was called when it came out, I didn't have to learn Objective-C.

Everybody was struggling with those square brackets and trying to understand why.

Why the hell having nil or nil

in, as a, as a pointer to know

why can we send messages to it and why the app is not crushing?

It's just undefined behavior.

I was I was okay with that.

I doing that for two years. Yeah.

And obviously there were macros engineers, out there, a lot of them,

but not nearly as many as,

as amongst people who wanted to get into,

engineering and engineering.

So, yeah, I was lucky enough to have this experience under my belt,

and I, I also had

quite some experience with mobile, at that point.

So iPhone SDK, I think it came out in 2009.

And I was doing macOS.

I also was doing, mobile, Windows Mobile.

Symbian was a joke.

I wrote apps for, for those,

obviously not in Objective-C, but it's still in mobile, right.

So, from the same ish area.

So I was aware of some of the constraints that you have to keep in mind

while developing for mobile. Yeah,

this is the mobile industry specifically for mobile engineering.

You will more so work towards consumer facing technology

like in a consumer facing domain, because I feel like as a backend engineer,

I've worked in B2B settings and then people learn

I actually want to work towards something that is more consumer facing.

I feel like if you're a mobile engineer, apps typically go to the consumers.

Yeah.

You know. Absolutely.

And honestly, I as a mobile engineer and I know

people are different and and mobile engineers are obviously different as well.

But for me it's very important to have, something that I work

on, have in front of people and a lot of people.

Right.

I definitely learned

my lesson when, I was working on an app

that was not really needed by anyone, and the company was developing it

just because they wanted to have presence on the App Store, because competitors do.

And it was tough to realize that you put so,

so much effort into building something that people don't need.

So every single company after that one,

they were very consumer facing

and moreover, they, their mobile app was the core of their business.

Yeah.

So you actively sought out for apps that made impact

for the consumers and were mobile was a core part of the business.

It's not that I saw that, but I actively reject companies

or don't start, conversations with companies that,

that don't do that. Exactly.

And that you still have that same mindset.

Like, that is the type of company in the industry you want to work in.

Absolutely. Yes. Yes.

So this is consumer face.

I understand that some B2B apps might also have the same.

Well, probably they won't say it has the same

scale, you know, in terms of the amount of people,

but in terms of usefulness that every employee,

of the company would have this thing installed

and they would open it regularly for whatever reason.

Right.

But if you have an app like workday on on your phone, right.

That's something.

Why would you have it on your phone?

They do have a mobile app.

They do.

And I did have it just to get push notifications when my,

when my vacation request was approved.

And that's pretty much it.

I like a messaging cube.

Yeah.

So that

again, not to,

not to say that it's not needed.

But for me, it is way more important.

Maybe behind the scenes.

The technology that is powering this app is amazing and it's super

interesting to work on, but for me, there is not enough motivation to,

to do work on on such kind of.

Yeah, yeah, I like that a lot.

Like, I feel like when it comes to an industry or a technology stack,

I don't have the same level of clarity yet where I go to a company and I say,

this is really what I want to be working on, and I feel like you had that.

You had that throughout the learnings, even though by chance

you were kind of the person that was brought forward

to learn about this technology.

And then through that, you came into mobile specifically.

You still found this industry or this type of company,

or you rejected any other, so there was no other option, basically.

Yeah. So I love that amount of clarity.

I think I the sooner you have that, the better it is for probably your,

your feeling of fulfillment.

Because then you can get to that position and you're just by virtue of you

being in that position, the position that fulfills you, you're motivated.

Absolutely. Yeah.

Yeah, I like that.

And Uber was a big part in that I'm assuming. Absolutely.

Because when it comes to their mobile presence, it's like all in.

Yeah ubiquitous. And

I think I

was lucky enough to, get into Uber in 2016.

Right before they started the big rewrite of, of their mobile app.

Yeah.

And, it is it was a huge undertaking.

You can imagine that for the company that is

that the mobile app is the core of their business.

The only thing that brings revenue, right?

And then all of a sudden you start to rewrite this thing from scratch.

It is a risky move.

And they wanted to do it very fast.

The initial estimation was to rewrite it in three months.

Millions of lines of code, hundreds of engineers,

mobile engineers,

yeah, was was crazy.

But then it gave me so much experience

with how how those kind of things could be navigated and,

and actually also understanding that rewrite is not always the answer.

Okay. Right.

What why is it so what did it not fix in the end?

What can you learn through that experience?

It fixed a lot that it broke a lot of people.

Oh in that sense, yeah.

I mean it's it's not even a joke because people go,

oh, okay, that's straight up left. Yeah.

But you were not one of these people.

That's another thing that

I have a very strong opinion

about when I try to separate,

my personal life and my work life.

Yeah.

And definitely when I get very involved into projects I can do over time, I.

It's not a big problem for me.

But I'm always aware where my line is, right.

I will not work during night time.

Like, yeah, even if I'm super into it.

I know that a future passion is going to regret this, so I'm not doing that.

And I saw a lot of people who burned themselves to the ground

just trying to push this thing out and understand probably that

I had the luxury of having my approach.

So the expense of this people.

Right, because they were doing work.

I don't know if you could be efficient in like 3 a.m.

in tonight, right? Yeah. But

on the other hand, the company would survive

if the project would be postponed for three more weeks.

Yeah, it's it would have been fine.

Yeah, exactly.

That's why I don't like the word deadline.

No one ever dies when the deadline has passed, basically.

So things will move on.

Yeah, probably.

You lose a lot of money, but. Yeah, that's that's.

Money is different, though. No one dies.

I hope otherwise. It's the wrong type of business to be.

Yeah, yeah, but talk to me about that experience specifically.

I'm assuming that

with an engineering culture, at least at the time that Uber had.

I know a lot of great engineers that come from that.

You're going to have very interesting discussions on what do we do,

how do we optimize or which decisions actually mattered.

Do you have a specific topic in mind

where you were like,

this is actually where I had a very strong opinion on this is what we need to do.

So from the times of the me right,

I probably cannot recall such things because I just joined the company.

Yeah.

And it was really funny because I was interviewing in April

and I specifically asked it was 2016,

so Swift was already there for, I think, a couple of years at that point.

But it wasn't mature. Right.

And I asked during the interview, they use Swift and the answer was no.

We are doing Objective-C 100%.

Swift is not mature enough.

It cannot support our scale.

And then

I joined 1st of June, and by the end of June,

we got a message from CTO saying we are rewriting this thing in Swift.

So that's quick.

That's really quick.

So yeah, that was that was fun.

But so that's for you to understand

that I was in the company for, for a month at the point where rewrites started.

And, at that point we were all hands, heads down,

actually executing, there was no time to

understand what's going on.

Yeah, exactly.

Well, yeah.

Any other specific time in your experience

at Google where you were like, okay, this was really an opinion that I had

that I had to push for because I'm curious how you did that.

So much, much later, after we did the rewrite of the of the app

and the team in Amsterdam, we were focusing mainly

on payments. And,

we saw a huge inefficiency in terms

of integrating of the, of the payment framework into new apps.

And Uber started to acquire new businesses more and more often.

So we had to integrate, the framework into new apps.

And long story short, we delivered the new new SDK,

which was much, much more efficient and pleasant to use.

But I had a big struggle convincing people that we have to migrate

existing usages of the payment framework or the old one onto a new one,

new approach, which was nicer, more efficient, allows for more,

monitoring, alerting and

whatnot for your specific use case, for example.

But since this doesn't bring any, any revenue

right there is you cannot put any money on these kind of migrations.

Metrics have to actually remain stable. Yeah.

For the migration to, to, to be success.

So you're net neutral. Exactly.

Yeah.

Or negative because you are actually investing engineering time into that.

It was really, really hard

to convince people that this is something we actually have to focus on, because

now we have two different ways of integrating payments into the app.

And then we explain engineers from other teams,

which way they should use.

Yeah, because they used to the old way.

They've been doing that for for years now.

What do we do?

So we had to actually go and educate people on

what money SDK is, how to use it, what are the benefits?

We had to sell this to other teams.

So they would either

themselves put migration onto the roadmap

or we would do that for them. Okay.

Which in the end turned out to be the vast majority of cases.

And then I had to convince our leadership that this is worthy,

that we have to do that. Gotcha. Yeah.

How long did that take from your idea of, okay, we have to do this

to eventually having people convinced that this is the way to go.

So the idea of, of the SDK itself, it was the end of 2019.

Yeah.

The implementation took, I think, a year. Okay.

Good year to actually hide all the complexity behind the nicer APIs.

First of all, to see what the nicer API is going to look like.

Yeah. And then hide it.

And after that, it was like 2021.

And the migration is still not done as far as I know.

Gotcha. So,

it's it's a

big organization, but yeah, it's really, really hard

to convince people, to do things and sometimes they use and abuse your,

your SDK in ways that make it a lot harder to maintain in the future.

Exactly.

And that's why I personally am a, a huge advocate of of hiding

as much API as possible and, making it private, basically

not not allowing to do anything from the outside as much as possible

and opening it up only if there is a strong use case.

Yeah, yeah, you get that.

That's really hard when it comes to then, because I'm now working in a landscape

where we're building a platform

and we're trying to integrate with a lot of mobile applications through SDK.

And they also have kind of a similar approach where not everything is exposed.

But yeah, circling back to what you mentioned, this was really a topic

where you would you compromise on this or you wouldn't really compromise?

No, I would not compromise on this. I knew that.

Well, first of all, that was my baby.

The the thing that,

that got started,

like I started from the well of a colleague.

We there were a bunch of people, but I was among

the people who started this thing from from, zero, essentially.

And I knew that this will bring

a lot of benefits.

Along the way and in the future,

and keeping two ways of doing

the same thing is not it is not is not sustainable.

And I so the thing is that I've been working,

at that point for five years at the company and.

Right.

And usually when something, when people from other teams

have questions, they would come to either me or, my colleagues.

And we had an influx of, of messages all the time.

Hey, how do I do this? Why is this not work?

And why is that?

Well, because you're doing the old stuff like migrate, please.

And then you wouldn't have those questions.

No. We need to move fast. We need to.

This is it. But we don't have a choice.

We don't have a choice.

And then we had to migrate this this case, and then all of a sudden, oh,

that was easier.

Like, it's so much nicer to use. Yeah.

All of a sudden.

Was this one of the reasons why in the end, you ended up leaving

because the migration is still not finished to this day.

You've moved on since.

Well, definitely not because of the might, not the single thing.

No, no, there was no,

one particular reason.

Yeah.

There were like a lot of little things.

For example, a stupid reason that

it was 20, 22.

So Covid just kind of started to end.

And I've been working from the year since,

in next to I'm still Station.

It was quite close to my house, so I had to bike like 20 minutes.

And Uber announced that they are opening this new

and shiny office, which they planned to move in 2024.

Yeah.

I was so far away,

got so annoyed that they, they started to brag about accessibility.

This office is so nice.

It's so close to

everything else.

Yeah, everyone else is.

So, and again, I'm not saying

that this is the reason, obviously, but, it was one

one of the things that tipped me over

and I wonder, it's been six years.

And, at that point when when they left and it's definitely

I, it felt like the right time to move on.

I found myself

in meetings.

I felt that I had no business with, like, why am I there?

Only because I knew how historically, things were evolving,

and how they were made and why they're made a certain way.

Yeah. Because of your knowledge and history.

Yeah.

Basically, I saw the payment framework built from the ground up

and then being migrated to to the next money SDK.

Yeah.

And I knew the reasons why certain things

were done in certain ways or shortcuts had been made.

So, that

I understood why I was needed or sometimes I didn't, and.

Fair enough.

Yeah.

But, yeah, it was definitely not something that, that

I was, very interested in.

And some people are completely fine with this kind of, kind of.

Work.

I would say.

But, for me, I love to build things.

I love to, to be an engineer.

Not like meeting engineer. Gotcha.

Yeah. You actually want to create? Yes.

Not just, makes sense.

I don't think I would be happy being a historian of the code

base, like, explaining why we have things,

but maybe once, maybe twice, but not continuously.

Like, as a daily or weekly thing.

And, you know, I found myself explaining certain things over and over again.

And this I definitely now see a lot more value than I saw back then, actually,

I seen, for example, a CEO of my new company

is repeating himself all the time,

and I understand, like, all of a sudden there is this light bulb.

They're doing this to make sure that they convey the message

that it's well received.

Because for me, it felt stupid.

I have already said this thing once.

It's like, why?

Why do I need to do it twice, thrice, four times?

But such an engineering mindset.

Yeah, I agree. Yeah.

Actually it is so, so useful because some details they

people don't pay attention because that's not their field.

That's not what they are interested in.

And then you have to put it out there

multiple times to make sure that it's actually understood.

This is what I really learned in product.

And maybe it's also the type of person

I am, I when it comes to product and why we do things.

I will take all the time you need.

I have all the patience to help you understand why we do things,

because I think from an engineering standpoint

that is incredibly valuable to understand why we do things

so you can execute better.

You can think along from a different perspective in product.

I really took as much time as we needed, and I mentioned that

every person in the team don't don't worry about asking again.

I will sit down and we'll explain and we'll go through it.

Also, because the domain was more complex, it was in sustainability.

It had to do with ESG and metrics and KPIs, and I think that is valuable.

I want that in my product person as well.

So, the fact that you see it in your CEO,

I think that's quite, quite admirable that they keep doing that.

Yeah, I think it's a great thing.

And understanding the way is, is actually it was a revelation for me.

And, you know, for me as, as an engineer, one of the most,

I wouldn't say powerful, but one of the,

one of the things that I'm not.

That I keep saying.

Yeah, I'm sorry, I don't understand.

It's something I don't understand.

I would call it a I call it out and ask the other person to explain to me,

that I actually actually understand,

if if that's something

that is out of my field and they would,

you know, go through some things that seem relevant,

but then I have no idea what they're talking about.

It's okay to not to understand something.

It's okay to say out loud and ask them to explain.

Because one of the things that people

actually love to do is to explain something they know.

Right.

And if you say I don't understand, they would love to explain this to you.

Yeah.

At least I hope so.

I've, I've also seen people

where they said I've already explained and then they lose it and I might

I really shouldn't be like this.

Like having an understanding

and also especially early on, I felt bad saying I didn't understand things.

And then I realized it's obvious I was a junior in the team.

Like, it makes sense for me to not understand things.

The worst thing I could do is saying I got it.

Well, I actually didn't get it. Yeah, I was like, that's that's even worse.

Hundred percent. Yeah.

You worked in payments at Uber.

Have you always worked in.

Because I think payments is quite a crucial domain

when we're talking about kind of app, it's how it makes money.

So probably needs to work quite well. Yeah.

Have you always worked in more crucial sides of the app.

There beyond as well.

It was mostly payments.

We did develop,

kind of React Native kind of thing framework.

Because so initial struggle

was the inefficiency that we saw that,

in different countries, drivers, whenever they want to,

fill in their bank

account requisites, whatever it's called,

they would have to fill in different forms in different countries.

Right? The regulations were different.

So it was initially done on the backend, the rendering part,

and it was done in code.

So basically you have to be an engineer to add

a new field onto this form.

And we wanted to make it easier.

So we wanted to,

actually build a system that would resemble React Native.

We were lucky enough

that was a person who actually built React Native and Facebook.

They knew, what what kind of inefficiencies that system had.

So they came with this knowledge

in at work, in the company and helped to shape this thing.

But for me personally, this journey ended pretty quickly.

I had to get back to, to payments.

I think

I got an offer I couldn't refuse a director.

So, Yeah, it was mostly payments.

Gotcha. Yeah.

Have you worked a lot with cross-platform technologies like React Native and stuff?

No, no, no,

no, I heard that TikTok open source, their framework, it's called links,

where they they fix some of the inefficiencies of React Native.

I did hear about that.

I didn't didn't try it.

Yeah. Yeah. Because I'm interesting.

Why has it been that you've always worked with the native

technologies, respectively?

Mainly because React Native doesn't feel native.

And that's something that we tried to fix with, with our framework.

Yeah. A tuber,

and I think which is pretty good result.

Like I say, we I was not in the team when,

when they

achieved, very good results with it.

Unfortunately, they struggled a lot with adoption.

The, the company, so they had to cancel the project.

But, the technology was amazing.

And for me, none of the back

then modern, cross-platform technologies felt native.

And I love beautiful interfaces.

I love snappy animations.

I love good interactions.

That would not look like Android

interface on iOS app or vice versa, right?

Yeah.

You sometimes you just have to go native. Yes.

There are parts of applications that are less important, less relevant.

So you can, but you still have to have them.

So you probably can't do them with with React Native and the like.

But you have to be very picky

if you want to deliver great user experience.

Do you think nowadays the landscape has evolved to where

cross-platform technologies have bridge that gap a little better?

Maybe.

But as I said, I didn't didn't dive too deep into it lately.

Yeah. Probably.

Yeah. Everything evolves. Right.

And I guess a lot of people, it's not only me who has this

idea that

cross-platform is actually not nice.

So, that yeah, I see that being fixed and, but I didn't didn't try it.

No. No worries.

I'm wondering, since you've really had this opinion of

I want to be at companies where mobile is core to their business,

do you see yourself striving away from that,

or is this really just the career path for you also towards the future

with AI?

Maybe you know, two months ago, not two months ago, half a year ago,

if someone would tell me that I would write Android code, I would be

in no way surprised.

And now look at me. I'm an Android engineer.

Yeah, but yeah, I'm joking, obviously, but,

Yeah, I actually started back in 2005.

I started as a backend engineer.

I, wrote in Java, and then the

destiny would

be on, on that path, of of mobile engineering, but,

yeah, I can definitely see doing whatever is needed.

And, you know, I'm in the luxury position of,

writing Android and learning new technology.

Yeah.

Well, being paid as a senior plus engineer,

I think this is also very important.

If you want to change your career path, you either have to,

you know, get a pay cut or you have to be in this luxury position

where the company trusts you enough that you can,

learn well, but be less efficient.

Yeah, learn and, you know, excel at this new field later on.

Yeah, absolutely.

As a last question, I still had in my mind it might be a little bit

of a personal one,

but you've been in the mobile field very hands on for a long, long time.

And now with AI that's changing.

Is it still fun for you, or has the joy kind of evolved in a different way?

You're less hands on typing, you're more orchestrating?

Has the joy changed?

It's just to fulfill you.

I think that's what brought me, brought, brought me some joy back.

I love learning new things.

I love doing something I've never done before, and that's why I'm here.

But, yeah, learning.

Doing Android.

While I have very little idea of what is

what is happening is actually very exciting.

It's actually really cool.

I find

this opportunity I'm really grateful for the company to to believe in me

and to as the, kind of cross-platform engineer, I feel,

because that's something that brought me this joy back.

In my previous company, I definitely didn't have this much fun,

as I do now, I love that.

Yeah, I love that.

Thank you so much for coming on and sharing

your not just your perspective, but also your career path.

I think there's a lot of interesting learnings for people

that want to become mobile engineers or are already on that path.

So thanks for coming on.

Thank you. Thank you. It's been a pleasure. Cool.

We're rounded off here.

If you're still with us, let us know what you thought

in the comment section below of this episode, and we'll see you in the next one.

Beyond Coding
