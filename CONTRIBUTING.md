# Contributing

Thanks for helping iOS developers get ready for iPhone Duo. This list is only useful if it's **correct**, so accuracy matters more than size.

## Adding a resource

1. Add it to the most specific section, at the end unless the section has an obvious order.
2. Use this format: `- [Title](https://link) - One-line description ending with a period.` Use the resource's real title.
3. Only add resources that are specific to iPhone Duo or foldables, and that you've read.
4. No affiliate links. No paywalled content without a free summary.
5. Open one PR per resource, titled `Add <name>`.

## Fixing an API name, availability or gotcha

The SDK is new and still changing. A correction PR must link to the source that confirms it, in this order of preference:

1. An Apple API reference page or documentation article (`developer.apple.com/documentation/…`)
2. A technote, or the Human Interface Guidelines
3. A tech talk, with a timestamp
4. An Apple Developer Forums answer from an Apple engineer

Third-party articles can point you to a change, but they aren't sufficient on their own.

## Adding a gotcha

Use the structure in [`docs/gotchas.md`](docs/gotchas.md): a short heading, one or two sentences on **why** it breaks, `// Before` and `// After` code where it helps, and a **Source:** line. If it can be detected statically, also open an issue on [FoldLint](https://github.com/divyaravitech/foldlint).

## Checks

CI runs [awesome-lint](https://github.com/sindresorhus/awesome-lint) and checks every link with [lychee](https://github.com/lycheeverse/lychee) on each PR and weekly.

Be kind. This project follows the [Contributor Covenant](https://www.contributor-covenant.org/version/2/1/code_of_conduct/).
