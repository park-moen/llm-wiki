# Visualizing Frontend Architecture

> Source: https://frontendatscale.com/issues/17
> Collected: 2026-08-24
> Published: 2024-03-03

How to draw frontend architecture diagrams that actually make sense.

ISSUE 17 / MARCH 3, 2024

Hey friends 👋 I made some solid progress on my Frontend Architecture Workshop these past few weeks. I’m excited to share more info about it soon, but today I wanted to give you a little sneak peek of something we’ll cover in more detail in the workshop—how to visualize our architecture using the C4 model.

Before we dive in, could you do me a quick favor and reply to this email with literally anything you want? I heard a few reports of the newsletter landing in the spam folder for some readers, and I learned that getting replies from you helps a lot with the deliverability. Here are a few ideas for what to include in your reply:

- “hi”, “hola”, or your favorite greeting in your native language.
- “my favorite JavaScript framework is _____”
- “I’d love to see a newsletter issue about _____” (I accept any requests except for recipes. I’m a terrible cook.)

Thanks so much in advance 🙏 Let’s jump into this week’s issue!

## SOFTWARE DESIGN

## Visualizing Frontend Architecture

Photo by Leah Newhouse on Pexels.com

One skill that I’m always trying to improve on (and that I highly recommend any software engineer to work on too), is the ability to draw clear, concise diagrams.

Architecture diagrams aren’t exactly hard to make—most are just boxes, arrows, and labels. But it can be challenging to draw them in a way that actually makes sense to whoever is reading them.

Different people like to see different things in a diagram. A product manager might be more interested in a high-level view of the architecture, while a frontend engineer would be more interested in how different UI components interact with each other. It’s hard to satisfy multiple audiences with the same diagram.

It would be dreamy if we had a set of diagrams that are both simple to make and allow us to zoom in and out of the codebase when needed, wouldn’t it? Well, dream no more, because that’s exactly what the C4 Model gives us.

## Maps of your code

Simon Brown, creator of the C4 model, likes to describe C4 as a way to create maps of your code. We can use these maps to orient ourselves (i.e. figure out where our application sits within the larger system), and to communicate our architecture to different audiences.

Here’s how Simon puts it on the C4 website:

> It’s a way to create maps of your code, at various levels of detail, in the same way you would use something like Google Maps to zoom in and out of an area you are interested in.

C4 gives us four fundamental diagrams to represent our architecture at different “zoom levels”. Here’s the shortest description of each one:

### Level 1: System Context Diagram

This is a high-level overview of our system, which allows us to take a step back and see the big picture. It shows our system as a big box in the center, along with the users and other systems it interacts with.

The System Context Diagram doesn’t contain any details about particular technologies or frameworks, which makes it ideal for communicating our architecture with non-technical people.

### Level 2: Container Diagram

Software systems are made of Containers, which are things like applications or data stores. For instance, the containers of an Internet Banking System might include a web application, a single-page application, a mobile app, an API, and a database.

Here, we’re starting to talk about technology choices, so the audience for this type of diagram is typically technical people.

System Context and Container Diagrams of the C4 Model

### Level 3: Component Diagram

Containers are made of Components (not to be confused with UI Components. More on that in a minute), which are the foundational building blocks of your application. Components could be things like classes, routes, or MVC controllers. We’ll see a more detailed example of a Level 3 diagram for a frontend application in the next section.

We’re getting closer to the code at this level, so these types of diagrams would make more sense for the engineers who with that particular application or database.

### Level 4: Code Diagram

Finally, if we zoom into a Component, we’ll see how it’s implemented as Code. At this level, we could use UML class diagrams, entity relationship diagrams, or dependency graphs to show how our classes, functions, and objects work together.

The level of detail of Code diagrams is rarely necessary, so it’s only recommended for documenting your more important or complex components. It’s also a good idea to generate these diagrams automatically from your source code using an IDE plugin or some other tool (like this one for VSCode.)

Component and Code Diagrams of the C4 Model

C4 also gives us some simple conventions about how to draw these diagrams, like how to label the boxes and arrows, as well as a set of complementary diagrams that we can use for particular scenarios, such as the deployment diagram.

Now that we have the maps of our code, let’s see how we can use them to navigate the joyful (but sometimes dangerous) waters of frontend development.

## Adapting C4 for Frontend Architecture

My favorite thing about C4 is that it doesn’t make any assumptions about a particular type of application, which makes it ideal for visualizing frontend architecture as well.

Level 1 and level 2 diagrams don’t have too many details about particular technologies, so we can use them as prescribed. For cross-functional teams, my recommendation is that you draw these diagrams in collaboration with other backend, data, and mobile engineers.

At levels 3 and 4, things get more interesting for frontend folks. When we zoom into a web app container, whether it’s a client-side single-page application or a “full-stack” app (e.g. built with a framework like Next.js or Laravel), we enter the magical realm of frontend architecture.

And you know what they say about the realm of frontend architecture… what happens in the frontend, stays in the frontend (unless it’s a network request.)

All joking aside (yes that was a joke, please don’t unsubscribe), I do think it’s worth adapting levels 3 and 4 of the C4 model slightly to better fit the needs of frontend applications. Here’s what I like to do:

### Level 3: Components -> Modules

One small tweak I like to make at this level is to replace the use of the word “Components” with “Modules.” I find that the word module is a bit less ambiguous than components, which in the frontend world typically mean a very specific thing—a UI component.

You don’t have to do this, of course. But if you’re adopting C4 on an existing application, chances are you already have your own terminology for some of C4’s abstractions, so know that it’s totally OK to adapt their names to better fit your needs.

The most significant adaptation I like to make, however, is to start letting the UI of our application guide how we break down our frontend container into modules.

For instance, take a look at the UI for this property management app called Nook, created by product designer Juliette Lagache. If we were building this app as a single-page application, here’s what the container diagram might look like (you might want to open the image in full screen to see it in detail.).

Level 3 "Module" Diagram for a Single-Page Application

I’m using screenshots of the different modules here for clarity, but these would typically be boxes with text in them.

In essence, modules are the top-level building blocks of your application. In this example, the dashboard, settings, and login screens would all be separate modules. They typically represent a top-level route in your app (like /dashboard, /settings, or /login) but this isn’t necessarily the case. For instance, the shopping cart of an e-commerce site might be its own module, even if it doesn’t have its own dedicated route.

### Level 4: Code -> UI Breakdowns

If we zoom into a module, we’ll start to see how it’s implemented as code constructs. If we were using object-oriented programming, we’d typically see a UML class diagram at this level. However, in modern component-based frontend development, these types of diagrams are not the most useful.

What I like to do instead in level 4 is to break down the UI of a particular module into individual components. This is what React popularized as Thinking in React, but I like to go a step further and categorize each component into one of three types: screens, features, and components.

- Screens — some modules are made of multiple screens. In the example below you’ll see that the Settings module has a series of tabs to navigate through Personal, Members, Integrations, and Billing settings. These would all be separate screens within this module (and likely, sub-routes like /settings/personal, /settings/integrations, and so on.)
- Features — these are essentially “large components.” It’s any part of the UI that is complex enough to justify a distinction from a base component. A rule of thumb I like to use is: if your component has so many subcomponents that it needs its own (nested) components folder to keep them organized, then it’s a feature.
- Components — these are the UI components we know and love. They can be big or small. Shared across your module or specific to a particular screen or feature. Buttons, tables, form elements, and icons all fall into this category.

Level 4 "UI Breakdown" of a module in our application

Regardless of which naming convention you use, it’s useful to have a simple hierarchy of components like this one to better understand where a particular component fits in your architecture.

The next question that comes up at level 4 is “How exactly do we translate this into code?” All of these components need to be wired up together somehow, and we need a folder structure to keep them organized. So, what’s the best way to go about it?

That’s a great question, and I’m glad you asked! Because that’s exactly what we’ll cover in the upcoming Frontend Architecture Workshop, which, as a reminder, will be completely FREE for Frontend at Scale subscribers.

So if that sounds interesting to you, stay tuned—more details about the workshop are coming very soon ✨

## ARCHITECTURE SNACKS

## Links Worth Checking Out

### 📕 READ

- Thinking about rewriting your codebase from scratch? Joel Spolksy thinks that’s the single worst strategic mistake a company can make, and he tells us why in this classic essay.
- Jake Archibald wrote a detailed article (full of interactive demos) about the most common issue when using the View Transitions API: handling aspect ratio changes.
- There’s a new React Hook on the block. It’s called useOptimistic, and not because he likes to see the glass half full. Sam Selikoff wrote a great blog post explaining how it works.
- Alex Kondov wrote a massive article about Clean Architecture in React with lots of great advice for building React applications using solid design practices.

### 🎥 WATCH

Diagrams as Code 2.0 by Simon Brown

Simon Brown is not only the creator of the C4 model but also the author of a diagramming tool called Structurizr, which we can use to model our architecture as code. In this talk at GOTO 2021, he shows us how to use the Structurizr DSL (domain-specific language) and the advantages of using a modeling language over a generic diagramming tool like Mermaid or PlantUML.

### 🎧 LISTEN

The Boring JavaScript Stack with Kelvin Omereshone

If you’re a fan of batteries-included frameworks like Laravel or Rails, but you wish you could keep using JavaScript for your backend, then I’m sure you’ll enjoy learning about the boring stack. It’s a full-stack stack built on top of Sails.js (an MVC framework for Node.js) and the Inertia library, which allows us to use React, Svelte, or Vue for the UI layer. This interview with Kelvin Omereshone on the Frontend Fire podcast is a great intro to this new stack.

That’s all for today, friends! Thank you for making it all the way to the end. If you enjoyed the newsletter, it would mean the world to me if you’d share it with your friends and coworkers. (And if you didn't enjoy it, why not share it with an enemy?)

Did someone forward this to you? First of all, tell them how awesome they are, and then consider subscribing to the newsletter to get the next issue right in your inbox.

I read and reply to all of your comments. Feel free to reach out on Twitter or reply to this email directly with any feedback or questions.

Have a great week 👋

– Maxi
