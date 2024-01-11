## Is smaller, more focused RAG still the way to win at scale?

At this point, most people who have been following generative AI discussions have probably heard of RAG (retrieval-augmented generation). For those who aren’t familiar, briefly it means you pull data from elsewhere and feed it through the LLM to get your final result. This is what we built at Earlyworm, a RAG system for turning online news and blogs into language learning material. 

Recently there’s been some doubt about the future of RAG. Context windows are bigger now (gpt-4-turbo has 128k, Claude 100k). OpenAI is releasing their Assistant API that manages RAG for you. Is there value in building your own RAG for your AI app anymore?

While larger context or managed RAG might work for the lowing hanging fruit applications like summarizing a long PDF or single-threaded chat applications, I see a lot of value still in the custom RAG development. 

AI apps have the following 4 distinguishing elements:

- Proprietary data (in training set and available for inserting to context)

- Exactness of response

- Cost

- Speed

Most startups aren’t going to have proprietary data, so that limits advantages to exactness of response, cost, and speed. In practically all of these, larger context windows and managed RAG lose out. 

***Exactness of Response***
Exactness of response is a moving target. Depending on who your user is, what time it is, what they are doing, etc. the definition of what is exact may differ. That means there’s no one size fits all generation technique. Unless the answer you’re looking for is a general summarization of a very long document, then bigger context windows are not going to give you a more exact response. Studies show that lazily packing irrelevant content into a large context window and relying on the LLM to figure it out performs worse than RAG.

***Cost***
It goes without saying that as far as LLM service costs go, when you’re paying for tokens then sending bigger contexts is going to be more expensive than smaller contexts. Managed RAG might be cheaper for applications with less traffic, but I am not sure and initially skeptical that the savings scale better than hosting your own RAG.

***Speed***
Sending more tokens means it takes longer to get a response. There are many studies that show even tens of milliseconds can influence retention and timespent at scale. For startups who want to differentiate by serving faster responses, using less tokens is going to be one of the easiest ways.

As an aside, one of my main workflows for ChatGPT is breaking a large project into tasks, and then smaller tasks until I can convert tasks to code. I get so much value from this workflow that I’m building some internal tools for Earlyworm to make it easier, if you’re interested in trying it out as well just send me an email. 

What’s interesting to me is the recursive nature of this workflow. It seems like this is not only how individuals operate but also how larger organizations operate. Vice Presidents set high level, abstract goals like “Release a record breaking product.” Directors and senior managers turn this into more concrete tasks based on their focus areas - like “Limit major bugs to x”, “Respond to customer messages in x minutes”, “Drive x% of sales through strategic marketing initiatives”, etc. Then managers and team leads break these into projects for teams of 3-5, like “Limit bugs by improving automated testing”, “Limit customer messages with improved help center material and onboarding”, “Increase sales through strategic partnerships with influencers”.

Within each project, each individual contributors break the major tasks into a series of smaller tasks, and on and on until the tasks are easy enough to focus on and complete in just a day or two.

To me, this looks something like LLMs with large context windows figuring out general strategy, LLMs with a little more focused dataset and maybe slightly smaller context windows putting together a sub-strategy, and then progressively focused LLMs breaking these into projects, tasks, and subtasks until finally the tasks are converted into the final products - code, emails, social media posts, marketing materials, etc.

While completely automated “AI agents” are appealing in a technical and magical sense, I highly doubt we are able to create such sophisticated pieces of machinery from non-deterministic LLMs. For the time being, I still see the best AI products being those that have a human-in-the-loop to course correct and guide the AI.
---------------
LLM-related reading from today, focused on how we get bigger context windows without needing to make new models. I don’t understand all of the math but with some work you can grasp the concepts and implications for people building on top of LLMs:

Tricks you can play to expand the context window of an LLM:

https://blog.gopenai.com/how-to-speed-up-llms-and-use-100k-context-window-all-tricks-in-one-place-ffd40577b4c

Good explainer on Rope: https://blog.eleuther.ai/rotary-embeddings/

A post about RoPE: https://medium.com/@ddxzzx/why-and-how-to-achieve-longer-context-windows-for-llms-5f76f8656ea9

Pinecone blogpost on why RAG isn’t dead: https://www.pinecone.io/blog/why-use-retrieval-instead-of-larger-context/

ALiBi: https://openreview.net/pdf?id=R8sQPpGCv0
