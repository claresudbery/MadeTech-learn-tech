<!-- Copilot / AI agent instructions for this repo -->
# Copilot instructions — BuggyProject

Purpose
- Assist maintainers by making small, well-tested edits to the Ruby code that power the tiny learning-topics site.

Big picture
- Lightweight Sinatra app serving a list of topics and the topic content.
- Core responsibilities are split between `lib/list_topics.rb` (discover topics) and `lib/view_topic.rb` (read topic README). The web entrypoint is `app.rb`.

Key files
- [app.rb](app.rb): Sinatra routes; calls into topic logic. Note: it currently calls `List.new` while the lib class is `ListTopics` (see "quirks").
- [lib/list_topics.rb](lib/list_topics.rb): builds topic list using `Dir.glob` and a local `Topic` value object.
- [lib/view_topic.rb](lib/view_topic.rb): reads `README.md` for a topic id.
- [spec/](spec): RSpec tests and `spec/topics_shared_context.rb` which creates temporary directories used by tests.
- [Makefile](Makefile), [Guardfile](Guardfile), [Gemfile](Gemfile): test & run workflows.

Workflows / useful commands
- Install: `bundle install`
- Run server: `make serve` (runs `bundle exec ruby app.rb -o 0.0.0.0`)
- Run tests (single run): `bundle exec rspec` — preferred for quick feedback.
- Continuous test/watch: `make test` (starts `guard`, which runs `bundle exec rspec` on changes).

Project-specific patterns & gotchas (important)
- Topic naming: topic directories are named with numeric prefixes like `01-First` and contain a README (tests create `README.md`).
- Inconsistent README matching: `lib/list_topics.rb` uses `Dir.glob("#{directory}/**/README")` (no `.md`) but tests and `ViewTopic` use `README.md`. Expect this to be a deliberate bug in the exercise — align changes with tests.
- Class / naming mismatch: `app.rb` calls `List.new`, but the class is defined as `ListTopics`. Changes to bridge these must preserve tests or update usages consistently.
- Regex parsing: `ListTopics::Topic` extracts `id`, `ranking`, and `title` using regex substitutions. Be careful when changing these — tests assert exact title/id formatting (see `spec/list_topics_spec.rb`).
- Tests use filesystem fixtures under `spec/tmp` created by `spec/topics_shared_context.rb`; prefer using these helpers rather than creating global temp files.

Editing guidance for AI agents
- Prefer small, test-driven edits. Run `bundle exec rspec` after changes.
- When fixing behavior, update all call sites: if renaming `ListTopics` to `List`, update `app.rb` and require statements in specs. Keep changes minimal and visible in tests.
- If touching file matching logic, make matching explicit (e.g. prefer `Dir.glob("**/README*", File::FNM_CASEFOLD)` or target `README.md`) and update tests accordingly.
- When modifying regexes in `lib/list_topics.rb`, add or update unit tests in `spec/list_topics_spec.rb` to document the expected output (title formatting, ordering, id values).

Examples from this codebase
- Ordering: topics are sorted by numeric prefix using `Topic#ranking` which returns `...to_i`. See `lib/list_topics.rb`.
- Reading content: `ViewTopic#execute(directory:, id:)` opens `"#{directory}/#{id}/README.md"` and returns `{ content: file.read }`.

When uncertain
- Run the test suite first (`bundle exec rspec`). Failing tests will indicate which behaviour is expected by the current spec suite.
- If you find mismatches (e.g. `README` vs `README.md` or `List` vs `ListTopics`) prefer to make the minimal change that makes tests pass and add a spec that captures the corrected behaviour.

If anything here is unclear, tell me which section you want expanded or which file you want me to change first.
