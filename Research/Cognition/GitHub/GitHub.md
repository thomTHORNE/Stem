Status: Open

# GitHub

An integration that brings a GitHub repository onto the board as a block: searching and navigating it, basic git operations, annotating a rendered document, and talking a document through with an AI.

## Settled

- An integration is a block whose content lives in an outside system.
- Content shown from the repository is a reference. Annotations written on it are original.
- Nothing is sent to GitHub or to an AI without informed consent.
- Whether it ships in v1 is decided in Scope.

## Open questions

- **Authentication** and where the credential is stored.
- **How git runs.** Commit, pull, and push from a desktop app: shelling out to an installed git, or a library inside the app. Each has platform consequences that need verifying before anything is stated.
- **Annotations on a changing document.** An annotation anchors to rendered text. When the file changes upstream, what happens to the anchor.
- **Scope of the block.** How much of a repository one block shows: a whole repo, a folder, a single document.
- **Images in a rendered document.** A README or doc may embed images. The canvas carries no raster content, and whether a rendered document inside a block is an exception — and whether its images are references like the rest of its content — is unresolved.

## Findings

None yet.

## Dependencies

- **Requires:** Cognition Blocks — an integration is a block. AI — conversing about a document.
- **Required by:** Recollection — repository content is indexed.
