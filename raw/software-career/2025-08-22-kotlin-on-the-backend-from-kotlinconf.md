# Kotlin on the Backend — What’s New From KotlinConf 2025

> Source: https://blog.jetbrains.com/kotlin/2025/08/kotlin-on-the-backend-what-s-new-from-kotlinconf-2025/
> Collected: 2026-10-21
> Published: 2025-08-22
> Format: Original official-page text, including collapsed FAQ answers, with layout markup cleaned and source links retained.

[Backend](https://blog.jetbrains.com/kotlin/category/backend/)
 [Community](https://blog.jetbrains.com/kotlin/category/community/)
 [KotlinConf](https://blog.jetbrains.com/kotlin/category/kotlinconf/)
 [News](https://blog.jetbrains.com/kotlin/category/news/)

# Kotlin on the Backend – What’s New From KotlinConf 2025

[Alina Dolgikh](https://blog.jetbrains.com/author/alinadolgikh)

KotlinConf 2025 has firmly placed server-side Kotlin in the spotlight, and rightly so. Between Spring collaboration, an upgraded ecosystem, performance boosts, AI tooling, and the adoption of the language by big companies, this is Kotlin at its (backend) best. Here’s a breakdown of what was announced, what’s new, and what you absolutely must check out from the recordings of the server-side talks at KotlinConf 2025.

The biggest news was traditionally covered in the keynote and followed up by talks over two days.

One of the revelations from the keynote that received many positive reactions was that half of Kotlin users are using the language for backend development. This isn’t a sudden shift but something we have observed over the years in the past [Kotlin Census](https://www.jetbrains.com/lp/kotlin-census-2020/) and [Developer Ecosystem](https://www.jetbrains.com/lp/devecosystem-2023/kotlin/#kotlin_what_for_2years) reports.

Check out why developers continue to choose Kotlin for the backend in 2025 in the teaser video linked below, in which engineers from five major tech companies share their Kotlin journeys.

https://www.youtube.com/embed/Ltl6VvyEv1I?feature=oembed

The full stories will be published soon on the [Kotlin YouTube channel](https://www.youtube.com/@Kotlin).

## A Strategic Partnership With Spring

The JetBrains and Spring teams announced a [strategic partnership](https://blog.jetbrains.com/kotlin/2025/05/strategic-partnership-with-spring/). The roadmap highlights include a complete null-safety guarantee for Kotlin and Spring apps, official Kotlin‑centric Spring tutorials and documentation, enhanced reflection performance using kotlinx.reflect, and evolving configuration DSLs.

Two sessions provided a deeper look into the world of Kotlin and Spring and its upcoming directions for development.

Rod Johnson, Spring’s original creator, spoke on why Kotlin with Spring delivers faster development with better maintainability. Spring Framework core committer Sébastien Deleuze presented updates in Spring Boot 4 that further refine Kotlin support, from annotation processing to coroutine integration.

https://www.youtube.com/embed/NcAW-FZtpzk?feature=oembed

https://www.youtube.com/embed/QExksqeNWbY?feature=oembed

## Ktor: Built for Modern Scalable Backends

Ktor continues to evolve into a powerful, modern framework for scalable applications. The recent update introduced built-in support for dependency injection with coroutine-based lifecycle management, and a new HTMX module for smoother server-driven UI development. Other notable additions include support for suspendable application modules, Unix domain sockets for the CIO engine, and a Gradle version catalog for easier dependency management.

Currently, the team is working on the API documentation, starting with [OpenAPI support](https://github.com/bjhham/open-api-eap-template).

For those working with Ktor (or planning to), these talks are must-watches:

? Talk | Presenter | Why It Matters |

[Simplifying Full-Stack Kotlin: A Fresh Take With HTMX and Ktor](https://youtu.be/AlGWsTXnWsY?si=2j9Yga-h4ptWD0xD) | Anders Sveen | Challenge the modern full-stack complexity and see how to build dynamic web apps with Ktor, HTMX, and kotlinx.html without SPA overhead. |

[Coroutines and Structured Concurrency in Ktor](https://youtu.be/JxTIZAEos8Y?si=x23sp67BOMSfIWvj) | Simon Vergauwen | Master robust and maintainable asynchronous patterns in real-world Ktor services. |

[Event-Driven Analytics: Building Real-Time Dashboards With Apache Flink and Ktor](https://youtu.be/2Eyq1VhBxUg?si=jZSoq2N_y5mq-LRv) | Viktor Gamov | Learn how to move beyond REST by using Apache Flink and Ktor to build real-time analytics dashboards with reactive, event-driven architecture.  |

## Kotlin and AI with Koog, Mellum, and Junie

JetBrains introduced a trio of AI tools designed for Kotlin development, including practical support for server-side tasks.

- [Koog](https://blog.jetbrains.com/ai/2025/05/meet-koog-empowering-kotlin-developers-to-build-ai-agents/) – a Kotlin-based framework designed to build and run AI agents entirely in idiomatic Kotlin.

- [Mellum](https://blog.jetbrains.com/blog/2024/10/22/introducing-mellum-jetbrains-new-llm-built-for-developers/) – the JetBrains’ in-house LLM, which is now open-sourced and fine-tuned for code generation and intelligent assistance.

- [Junie](https://www.jetbrains.com/junie/) – the JetBrains AI coding agent integrated in your favorite IDE and able to target complex Kotlin development across server, mobile, and web domains. It’s now [available in GitHub](https://www.jetbrains.com/junie-github/)as part of the Early Access Program. Join the waitlist to try it for free.

? Talk | Presenter | Why It Matters |

[Building AI Agents in Kotlin with Koog](https://youtu.be/O8WQCrdza8E?si=Gl_1Cgd1C1JZ2_wD) | Vadim Briliantov | An overview of AI agents, their building blocks, workflows, and how to build them effectively in Kotlin from the JetBrains AI Agents Platform tech lead. |

[Kotlin’s Gam[e]bit: LLM-less AI for Board Games](https://youtu.be/434WFSiYj9k?si=R6EGZEjM-aR1xU09) | Dmitro Kurets | Discover how Kotlin can power AI without relying on large language models. |

[From 0 to h-AI-ro: high-speed track to AI for Kotlin developers](https://youtu.be/zBgb0z1pQkM) | Urs Peter | This talk gives you a clear and practical understanding of key terminology, concepts, and frameworks relevant to the Kotlin ecosystem in the world of AI |

[Building an Agentic Platform with Kotlin: Powering one of Europe’s Largest LLM Bot](https://youtu.be/mQhtLCMiUMA?si=IHNPlV4-6LJ43aqY) | Patrick Whelan | A look at some of the design decisions of the Arc framework, an open-sourced Kotlin-based AI framework, which will provide you with an understanding of the challenges in building an LLM application at this scale |

[LangChain4j With Quarkus](https://youtu.be/hxEM5J6QfLY) | Max Rydahl Andersen, Konstantin Pavlov | The talk explores tools integration, dependencies management, and Kotlin’s idiomatic features, helping to simplify AI workflows. |

## A New Milestone: Exposed 1.0

After substantial restructuring, Exposed 1.0 ([now in Beta](https://github.com/JetBrains/Exposed/releases/tag/1.0.0-beta-4)) delivers on its promise of a type‑safe, expressive, and idiomatic Kotlin SQL library.

With added support for full R2DBC, enabling non-blocking database operations through suspending functions and flows, new SQL features, improved onboarding materials, and a new IDE plugin, it removes friction from database operations while enhancing safety and maintainability.

Chantale Loncle’s talk dives into the details of what’s new and what’s next. Check it out below.

https://www.youtube.com/embed/xgfeqj8UyVA?feature=oembed

## Win With Kotlin Notebook

Roman Belov, the lead of Kotlin Moonshots projects, shared a fun yet practical example of Kotlin Notebook in action: finding the longest possible sailing route in 24 hours without repeating waypoints. With Kotlin Notebook, even complex logic becomes intuitive and visual. Watch the talk to learn the full story!

https://www.youtube.com/embed/-CznWwKD-WE?feature=oembed

## http4k: Pure Kotlin, Protocol Agnostic

The talk Full Stream Ahead: Breaking Protocol Barriers With http4k showcased a fully functional, coroutine-based approach to server-side development that sidesteps servlets entirely. It emphasized composability, lightweight design, and streaming I/O with functional purity.

https://www.youtube.com/embed/vewgb-vyJME?feature=oembed

## Other Server-Side-Related Talks

? Talk | Presenter | Why It Matters |

[That’s Unpossible – A Full-Stack Side Project Webapp (Including a High-Fidelity UI!) All in Kotlin](https://youtu.be/FXGT6HbBXNw)
 | Dan Kim | A practical walkthrough of building a fully functional web app entirely in Kotlin, from backend to high-fidelity UI.
 |

[IoT Development With Kotlin](https://youtu.be/bKwdgMKyFOY) | Erik Hellman | A hands-on look at building IoT applications using Kotlin, with key technologies and integration strategies. |

[Taming Asynchronous Beasts: Debugging and Performance Tuning in a Coroutine World](https://youtu.be/QTX5_JV4TVU) | Marcin Moskała | An expert-level deep dive into coroutine debugging and performance tuning from the author of Kotlin Coroutines: Deep Dive.
 |

[Kotlin Multiplatform’s Cross-Platform Brilliance at Norway’s 377-Year-Old National Postal Service](https://youtu.be/OuX5325yq_I) | Anshika Koul | A real-world case study of Posten Bring, detailing how the national postal service used Kotlin on all platforms to modernize logistics at scale. |

[Kotlin Clean Architecture for Serverless: Business Logic You Can Take Anywhere](https://youtu.be/6jZa4B-If-I) | Elena van Engelen | A clean architecture approach to structuring serverless Kotlin apps, focused on portable business logic across cloud providers. |

[From Data to Insights: Building a Bluesky Bot powered by AI](https://youtu.be/uGgK0F3cLEI) | Raphael De Lio | How to quickly collect, process, and analyze data using Kotlin – demonstrated via a Bluesky bot that extracts insights from real-time streams. |

That wraps up our curated list of backend-focused talks at KotlinConf 2025. Explore them all via the [conference website](https://2025.kotlinconf.com/talks/) or mobile app, and share your favorites with the community.

## Kotlin’s Backend Momentum

KotlinConf 2025 made it unmistakably clear: Kotlin continues to thrive in server-side development. It’s pragmatic, loved by engineers, and increasingly adopted across infrastructures and domains from Spring and Flink to Kafka and serverless platforms. Much of this progress is powered by exceptional tooling from JetBrains, including intelligent IDE support and new AI integrations.

That being said, the biggest engine behind Kotlin’s evolution is the community: library maintainers, open-source contributors, companies scaling Kotlin in production, and speakers sharing hard-earned lessons. Thank you all very much, we wouldn’t be where we are without you!

If you’re evaluating Kotlin for backend development in 2025, now’s the time to give it a serious look. And if you’re already building with Kotlin – thank you! You’re shaping the future of a language that grows stronger with every contribution.

[Server-Side](https://blog.jetbrains.com/kotlin/tag/server-side/)

- Share

- [Facebook](https://www.facebook.com/sharer.php?u=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2025%2F08%2Fkotlin-on-the-backend-what-s-new-from-kotlinconf-2025%2F)

- [Twitter](https://twitter.com/intent/tweet?source=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2025%2F08%2Fkotlin-on-the-backend-what-s-new-from-kotlinconf-2025%2F&text=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2025%2F08%2Fkotlin-on-the-backend-what-s-new-from-kotlinconf-2025%2F&via=kotlin)

- [Linkedin](http://www.linkedin.com/shareArticle?mini=true&url=https%3A%2F%2Fblog.jetbrains.com%2Fkotlin%2F2025%2F08%2Fkotlin-on-the-backend-what-s-new-from-kotlinconf-2025%2F)

[Prev post Kodee’s Kotlin Roundup: 2.2.20-Beta2, Kotlin LSP Update, Shipaton 2025, and a New Face on the Foundation Board](https://blog.jetbrains.com/kotlin/2025/08/kodee-kotlin-roundup-july-25-edition/)[What’s Next for Kotlin Multiplatform and Compose Multiplatform – August 2025 Update Next post](https://blog.jetbrains.com/kotlin/2025/08/kmp-roadmap-aug-2025/)

#### Subscribe to Kotlin Blog updates

Thanks, we've got you!
