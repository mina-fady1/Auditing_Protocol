# Auditing-Protocol

A GitHub repository template for SanadSquad audit engagements. Repositories
created from it receive the vulnerability report template and a consistent,
client-ready issue-label taxonomy.

## Use this template

1. In GitHub, select **Use this template** and create the new audit repository.
2. The creation event is a `push`, so the copied setup workflow runs on the
   initial commit. If GitHub asks for a first commit during creation, make it.
3. Wait for **Set up Auditing Protocol** to complete in the Actions tab.
4. Open **Issues → New issue** and select the vulnerability template.

The new repository receives:

- `.github/ISSUE_TEMPLATE/vulnerability.md`, the SanadSquad vulnerability
  report template;
- exactly the 15 required SanadSquad labels, with their specified names,
  colors, and descriptions; and
- removal of every pre-existing label that is not one of those 15, including
  GitHub's normal default labels.

> The workflow intentionally treats labels as the template-owned set during
> initialization. Do not add custom labels until this one-time workflow has
> completed, because any pre-existing non-SanadSquad labels are removed.

## One-time initialization

GitHub templates copy files but do not copy labels. The copied workflow listens
for `push`; GitHub emits that event when a repository is generated from a
template. It explicitly skips any repository named `Auditing-Protocol`, so the
source template is never changed.

The workflow first reads its own state through the Actions API. It lists all
labels, deletes labels outside the required set, then creates or updates the
15 required labels. It finally calls GitHub's **disable workflow** API on
itself. A disabled workflow cannot be triggered by later pushes, while its YAML
remains in the repository as an auditable record. This avoids relying on a
running job to delete the workflow file that defined it.

The per-repository concurrency group serializes overlapping pushes. If setup or
disabling fails, the workflow is left active and a later push retries the
idempotent label setup. If a run was already queued when disabling occurred,
the workflow-state check exits without changing labels.

The workflow uses only the built-in `GITHUB_TOKEN`, with `issues: write` for
labels and `actions: write` to disable itself. No SanadSquad credential or
personal access token is needed. The destination repository must allow these
`GITHUB_TOKEN` write permissions; a personal test repository can be configured
at **Settings → Actions → General → Workflow permissions** if an account or
organization policy restricts them.

## Updating the template

Maintain the template source as follows:

- edit `.github/ISSUE_TEMPLATE/vulnerability.md` to change the report form;
- edit the `required_labels` array in
  `.github/workflows/setup-auditing-protocol.yml` to change labels; and
- edit the workflow itself only after considering its `push` trigger, template
  safety check, and self-disable step.

Changes to this template do not propagate to repositories already created from
it. Update those repositories separately. To intentionally run initialization
again in a generated repository, re-enable the workflow in the Actions UI and
push a commit; this will again remove non-template labels.

## Testing

Local validation covers the repository structure, the exact issue-template
body, label count and values, and YAML syntax. GitHub-hosted integration tests
require creating a temporary repository from this template in a personal GitHub
account; they cannot be run from this local workspace or without access to the
target organization.

For the full integration check, create `Auditing-Protocol-Test` from this
template and verify:

1. the three copied files are present;
2. **Issues → New issue** offers the vulnerability template with its expected
   body;
3. **Issues → Labels** contains exactly 15 labels with the required names,
   colors, and descriptions, and no GitHub defaults such as `bug` or
   `documentation`;
4. the setup workflow is shown as disabled after its successful run; and
5. a later commit does not create another initialization run.

Also confirm that the original repository named `Auditing-Protocol` has no
label changes: its job is skipped by the explicit repository-name guard.
