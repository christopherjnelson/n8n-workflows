[Screenshots](https://imgur.com/a/WdgifeB)

**A Little Background & The Idea:**
About 12 years ago, I ran a medium-sized Marvel Cinematic Universe News Site (MCUExchange). It gained some traction, but eventually, the effort led me to sell it and focus on mobile/web development. Fast forward to today, I'm diving into n8n and Langchain, and recreating that site concept through automation seemed like the perfect project!

**Workflow Evolution:**
I initially jumped in using mostly LLMChains and Code nodes because that felt fastest to get *something* working. From there, I gradually worked backward, replacing the code with built-in or community n8n nodes. The goal was to end up with a more transferable, low-code solution that others could potentially adapt just by changing a few initial parameters.

**So, What Does It Do?**
This workflow automates content creation by monitoring news articles posted to specific subreddits (like `/r/marvelstudios`, filtering by the "Article" flair).


**Here's the gist:**

* It finds relevant Reddit posts linking to external articles.
* It extracts the article content from the linked page.
* It grabs and summarizes Reddit user sentiment from the comments on that post.
* It then uses AI agents (representing different writer styles) to generate a new article draft based on the extracted content and comment summary.
* The final article gets SEO tweaks (headline, basic formatting, meta description) before being posted to WordPress.


To support this, I'm using Supabase in two ways: first, as a vector store to build a knowledge base as articles are published. This helps give the AI writers context beyond their training data cutoff. Second, it acts as a simple table to check for duplicate post IDs, ensuring we don't process the same article twice. *(Side note: I tested this approach on `/r/dc_cinematic` filtering by the "News" flair, and it worked there too)*. Discord nodes are also sprinkled throughout to send progress updates to a channel.

**Tech Choices:**
I've been mostly using DeepseekV3 via OpenRouter – it's cost-effective and seems quite good at following instructions. For simpler tasks, Google's Gemini Flash 2.0 works well. I did include some auto-fixing parser nodes, which were helpful when testing smaller models like Mistral-Small (which *did* work, albeit slower, hinting that local execution on decent hardware is feasible). For reference, it cost less than a dollar via open router to write 100 articles.

**Usage & Results:**
To populate the site initially, I ran the workflow in batches (fetching 10 posts, then 20, etc., up to 100) to get articles going back a couple of months. Theoretically, you could go much further back to really build out content. For ongoing use, once I set up a production environment (likely Digital Ocean), I'll probably schedule it to run hourly, checking for maybe the latest 20 "Article" posts.

**Current Limitations & Caveats:**


* **Image Posts:** Many subreddits have news shared as images (screenshots of tweets, etc.). The workflow currently filters these out, meaning it misses potentially important news. I experimented briefly with OCR but haven't fully cracked this yet.
* **Other Flairs:** Specifically on `/r/marvelstudios`, there's a "Promotional" flair for trailers, official merch, etc., which this workflow also misses. Finding source links for these might require a different approach (maybe vision models + search).

**Future Ideas / Todo:**


* **Featured Images:** This is manual right now. Automating finding or generating *relevant* featured images (like actor headshots + character images) is tricky, especially ensuring quality and relevance. ComfyUI locally works okay, but integrating it smoothly or finding a reliable API-based solution needs more thought.
* **WordPress Tags:** The SEO node suggests keywords, but I haven't automated adding them as tags in WordPress yet. Ideally, it would check existing tags, use them, or create new ones. I wanted to manually tag the first batch anyway, but this is a clear next step for automation.
* **"Editor-in-Chief" (Fact-Checking):** My initial attempt at an EIC agent with web search tools often struggled with very new information or got stuck in loops. For now, giving the writers access to the continuously updated Supabase KB felt more reliable. I might revisit a separate EIC/review workflow later.
* **Opinion Pieces:** I plan to scrape Reddit comments from top posts (any flair) and have the LLM try to come up with opinion pieces (or unpopular opinion pieces!) based on what the community is talking about that week and post them gradually throughout the week. This would give the site both news and “editorials”. I’ll likely just create a new workflow to manually run once a week as adding all of that logic to the current one would just look messy.

**Resources**

* [Supabase Setup](https://supabase.com/docs/guides/ai/langchain?queryGroups=database-method&queryGroups=database-method&database-method=sql)
* [Discord Setup](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.discord/)
* [ScrapeNinja Setup](https://scrapeninja.net/docs/n8n/)
