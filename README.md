# Auditing-Protocol

A GitHub repository template for SanadSquad audit engagements. Repositories
created from it receive the vulnerability report template and a consistent,
client-ready issue-label taxonomy.

## Set up the template repository (one time, admins only)

Do this once, when publishing the template itself:

1. Create the repository in the SanadSquad organization named exactly
   `Auditing-Protocol` (hyphen, not underscore). The setup workflow skips any
   repository with this name, so a different spelling weakens that safeguard.
2. Open **Settings → General** and tick **Template repository**. Without this,
   the repository does not appear in the **Repository template** dropdown, and
   the workflow's `is_template` guard is not active.
3. Do not run the setup workflow here. If it is ever triggered manually, it
   skips itself in this repository.

## Use this template

1. In GitHub, select **Use this template** and create the new audit repository.
2. The creation event is a `push`, so the copied setup workflow runs on the
   initial commit. If GitHub asks for a first commit during creation, make it.
3. Wait for **Set up Auditing Protocol** to complete in the Actions tab.
4. Open **Issues → New issue** and select the **Vulnerability Report**
   template.

> **If the workflow does not appear in the Actions tab after about a minute,**
> GitHub occasionally fails to emit the `push` event for a brand-new repository
> created from a template. Either push any commit (for example, an edit to this
> README) or open **Actions → Set up Auditing Protocol → Run workflow**. The
> workflow stays active until it succeeds, so retrying is always safe.

The new repository receives:

- `.github/ISSUE_TEMPLATE/vulnerability.md`, the SanadSquad vulnerability
  report template. It has a short YAML front matter block (`name` and `about`)
  so GitHub lists it on the **New issue** page, followed by the report body;
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
template. It skips the source template repository in two ways: it does nothing
if the repository is marked as a template, and it does nothing if the
repository is named `Auditing-Protocol`. Both checks exist so the source
template's labels are never changed, even if one of them is misconfigured.

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

- edit the body of `.github/ISSUE_TEMPLATE/vulnerability.md` to change the
  report form, and keep its front matter (the `---` block at the top) intact,
  because GitHub requires `name` and `about` to show the template;
- edit the `required_labels` array in
  `.github/workflows/setup-auditing-protocol.yml` to change labels; and
- edit the workflow itself only after considering its `push` trigger, template
  safety checks, and self-disable step.

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
2. **Issues → New issue** lists the **Vulnerability Report** template, and
   opening it shows the expected body;
3. **Issues → Labels** contains exactly 15 labels with the required names,
   colors, and descriptions, and no GitHub defaults such as `bug` or
   `documentation`;
4. the setup workflow is shown as disabled after its successful run; and
5. a later commit does not create another initialization run.

Also confirm that the original repository named `Auditing-Protocol` has no
label changes: its job is skipped by the template-repository and
repository-name guards.