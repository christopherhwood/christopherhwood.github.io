I worked on a lot of personal AI projects in 2024, and have decided to open source all of them. I think the work done in a lot of them is pretty interesting and there may be some novel insights gained by looking at the code. I will try to call out interesting parts briefly here but I have more information about each project on my website (linked below for each project) and those also include videos of demos of each product.

In order of my opinion of most interesting to least interesting (not to say any are not interesting...):

## ui-builder (mid-2024)

Github: https://github.com/EarlywormTeam/ui-builder

Details: https://portfolio.christopherhwood.com/llm-drag-drop-website-builder

This is a drag and drop website builder with a few twists. An LLM writes all of the frontend state management code, and the LLM can help you to redesign or beautify your site. It has some novel ways of switching between an edit and preview modes that uses some potentially unsafe javascript eval in your browser, but enables some very cool features. One of which is live editing the code in a small window, which is far more convenient than toggling buttons or moving sliders to do things like change colors or adjust font sizes. Instead, just edit your component's tailwind classes. It uses tailwind, React, and shadcn components.

## react-component-agent (early 2024)

Github: https://github.com/christopherhwood/react-component-agent

Details: https://portfolio.christopherhwood.com/frontend-component-builder-agent

An AI Agent that builds your frontend components. It first formulates a plan which you can see appear in the interface, then goes step by step building the component. As it completes steps, there is a live demo view of the component available for you to play with and a view of the code it has produced on the right side of the page as well. It has some special tricks for building components that use contenteditable. As of early 2024, it worked fairly reliably and had surprisingly good results but it was expensive to run and took a while. I'm sure that would be improved with today's LLMs.

## agent2 (early 2024)

Github: https://github.com/christopherhwood/agent2

Details: https://portfolio.christopherhwood.com/node-js-code-generator

This is a poorly named project - it is actually a Node.js code generator application. It works with existing repositories similar to how Devin works (but created several months before Devin was announced), and produces a ton of artifacts besides just the code change pull request. It also spins off PRDs based on the task, technical design docs, coding style guides, and more report style artifacts that it uses internally. It worked well enough to start improving itself although the reliability was a bit questionable and like most LLM projects in early 2024 it was slow and expensive.

## qckfx-ads (late 2024)

Github: https://github.com/EarlywormTeam/qckfx-ads

Details: https://portfolio.christopherhwood.com/ecommerce-image-generator

This was actually launched and used by others for a bit. It uses ComfyUI workflows to generate images that contain products from ecommerce brands.It was interesting working with ComfyUI and all of the image manipulation tools out there. I was able to generate pretty high quality product images straight out of the flux model without having to "photoshop in" the product photo, which blew my mind. It was not super robust though.

## What Are You Doing Now?

I recently founded qckfx.com, where we're building the first fully-automated bug-fixing AI. Our agents automatically detect and fix bugs from your task trackers, delivering complete pull requests with fixes, tests, and root cause analysis - no human intervention (or prompting) needed.

If this is something you're interested in, you can learn more at qckfx.com or feel free to shoot me an email at chris.wood@qckfx.com. 

We're looking for beta users and hiring founding engineers (preferred experience with AI agents, AI codegen, AI workflow automation, machine learning, or AI evals).
