# Ideas — photos

<!-- Use this file to capture feature ideas, plans, and future work.
     Dictate here via voice, jot quick notes, or write detailed specs.
     During a Claude Code session, say "implement the plan in [[ideas]]"
     and Claude will read this file and get to work. -->

## Feature ideas
- make a periodic script that runs automatically that pings the supabase cluster in some way to keep it from getting archived



## Improvements
<!-- Things that could be better. -->

## Technical debt
- **True testing / QA setup**: Add a test environment with dedicated test photos, a separate Supabase instance (QA-like), and the ability to run the full app locally so new functionality can be developed and validated without touching real data. Likely involves a `.env.test` / `.env.local` override, a seeded R2-compatible local bucket (e.g. Minio or R2 dev), and a separate Supabase project or local `supabase start` stack.

## Experiments
<!-- Things to try, spikes to run. -->
