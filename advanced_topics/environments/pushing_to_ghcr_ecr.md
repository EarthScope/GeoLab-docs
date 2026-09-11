# Pushing Images to GitHub or Amazon Image Repositories

Step-by-step guides to authenticating and pushing Docker images to the Amazon ECR Public Gallery and GitHub Container Registry (GHCR). Each section covers login, tagging/pushing, and troubleshooting authorization errors that are frequently encountered.

## GitHub Container Registry (GHCR)

A guide to authenticating to GitHub Container Registry with a personal access token (PAT) or the GitHub CLI, pushing images, and troubleshooting common authorization errors.

> **Note:** GHCR images are namespaced under a GitHub user or organization, e.g. `ghcr.io/<owner>/myrepo`. Package visibility (public/private) and access are managed per-package in GitHub, separately from the repository the code lives in.

### Prerequisites

The following software needs to be installed.

- Docker Desktop installed and running.
- A GitHub account with permission to push packages to the target owner/organization.
- Either:
  - A **personal access token (classic)** with the `write:packages` scope (and `read:packages`, `delete:packages` if needed), or
  - **Optional:** The **GitHub CLI** (`gh`) installed and authenticated

### Create a Personal Access Token (One-time)

In GitHub select: **Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**.

Select these scopes:

- `write:packages`: required to push
- `read:packages`: required to pull
- `delete:packages`: optional, only if you need to delete versions

Save the token in a safe location, because GitHub only shows it once.

> Alternatively, skip token management entirely and use `gh auth login`, then `gh auth token` to retrieve a token for the login step below.

### Log in to GHCR

Using a Personal Access Token (PAT):

```bash
export CR_PAT=<YOUR_TOKEN>
echo $CR_PAT | docker login ghcr.io -u <github_username> --password-stdin
```

Or, using the GitHub CLI's token:

```bash
gh auth login
gh auth token | docker login ghcr.io -u <github_username> --password-stdin
```

A successful login prints `Login Succeeded`.

> **Note:** Classic PATs don't expire unless an expiration date is set when creating them. If an expiration date is set, a new token needs to be generated when it expires.

### Tag and Push an Image

Rename or `tag` the image with the GitHub Container Registry address, GitHub username, the image name, and tag.

```bash
# Tag your local image with the full GHCR URI
docker tag my_geolab:latest ghcr.io/<github_username>/my_geolab:latest
```

Publish or `push` the image to GHCR.

```bash
# Push it
docker push ghcr.io/<github_username>/my_geolab:latest
```

Replace `<github_username>` with your GitHub username or organization name. The first push to a new name automatically creates the package; you can adjust its visibility (public/private) and linked repository afterward in **Package settings**.

### Set the Repository to Public

Images pushed to GHCR are private by default. The image must be set to public for GeoLab to access it. These steps change a GitHub Container Registry (GHCR) package's visibility so the image can be pulled without authentication.

**Steps:**

1. Navigate to your packages:
   - **User-owned:** `https://github.com/users/<github_username>/packages`
   - Or go to your profile or org page and click the **Packages** tab.

2. Select the container image (package) you want to change.

3. On the package page, select **Package settings** (right-hand sidebar or the gear icon).

4. Scroll down to the **Danger Zone** section.

5. Select **Change visibility**.

6. In the dialog, select **Public**.

7. Type the package name to confirm, then click **I understand the consequences, change package visibility**.

The image is now publicly accessible without authentication.

### Troubleshooting GHCR

A 401 Unauthorized or 403 Forbidden are the two most common errors when pushing an image to GHCR.

#### 401 Unauthorized

You are **not authenticated**: the token is stale, expired, lacks scope, or was never provided to Docker.

1. **Regenerate or refresh your token.** Classic PATs with an expiration date need to be re-created once they lapse.
2. **Confirm the token has `write:packages` scope.** A token missing this scope will authenticate but fail on push.
3. **Re-run the login step** with a fresh token:

   ```bash
   echo $CR_PAT | docker login ghcr.io -u <GITHUB_USERNAME> --password-stdin
   ```

4. **Check the username matches the token's owner**: a mismatched `-u` value is a common cause of 401s.

> A mid-session 401 usually means the PAT expired or was revoked. Generate a new one and log in again.

#### 403 Forbidden

You **are authenticated**, but you lack permission, or the package's access settings are blocking you.

1. **Verify token scopes** include `write:packages` (and `read:packages` for pulling).
2. **Check package visibility and access.** For an existing package, go to its **Package settings → Manage Actions access / Invite teams or people** and confirm your account has write access.
3. **Confirm the namespace is correct.** Pushing to `ghcr.io/<OWNER>/...` where `<OWNER>` is an organization requires that the organization has enabled package creation for your role, and that you're a member with sufficient permissions.
4. **If using GitHub Actions**, make sure the workflow has `permissions: packages: write` set, since the default `GITHUB_TOKEN` scope is otherwise read-only.

#### Clearing credentials and starting fresh

If refreshing the token doesn't fix a 401/403, clear cached Docker credentials and log in again from scratch.

**Log out of GHCR and clear stored credentials:**

```bash
# Log out of the registry
docker logout ghcr.io
```

This removes the entry from Docker's credential store (`~/.docker/config.json`). To clear **all** stored Docker credentials, not just this registry:

```bash
# Inspect first, then remove
cat ~/.docker/config.json
rm ~/.docker/config.json
```

**Revoke and replace the token:**

- In GitHub, go to **Settings → Developer settings → Personal access tokens** and delete the old token.
- Generate a new one with the correct scopes (see Step 1).
- If you used `gh auth login`, run `gh auth logout` first, then `gh auth login` again.

After clearing both, re-run the full login sequence from the **Log in to GHCR** step:

```bash
echo $CR_PAT | docker login ghcr.io -u <GITHUB_USERNAME> --password-stdin
```

#### GHCR Quick reference

| Error | Meaning | First thing to try |
|-------|---------|--------------------|
| `401 Unauthorized` | Not authenticated / token expired or wrong scope | Regenerate token, re-run `docker login` |
| `403 Forbidden` | Authenticated but not permitted | Check token scope and package access settings |

#### Token lifetimes at a glance

- **Classic PAT:** No expiration by default, unless one was set at creation
- **`gh auth token`:** Tied to your `gh` CLI session; refresh with `gh auth login` if it stops working

When a token expires or is revoked, re-authenticate from the relevant step above.

---

## Amazon Web Services Public Elastic Container Registry (ECR)

A step-by-step guide to authenticating and granting Amazon ECR credentials to Docker for pushing images to ECR, plus troubleshooting the two common authorization errors.

> **ECR Public vs. private ECR:** Public repositories live at `public.ecr.aws/<REGISTRY_ALIAS>` and use the `aws ecr-public` command set. Authentication **must** be done in `us-east-1`, regardless of where you or your users are located.

### Prerequisites

- AWS CLI v2 installed (`aws --version`).
- AWS profile [configured](https://docs.aws.amazon.com/cli/v1/userguide/cli-configure-files.html).
- Docker installed and running.
- A public registry alias (find it in the ECR console under **Public registries**, e.g. `a1b2c3d4`).

### Create an AWS ECR Public Repository

Create an ECR public repository to store your images. If you don't have a repository, AWS will create one, but it's better to choose your own repository name. Choose a name to replace `<my-public-repo>`.

```bash
aws ecr-public create-repository \
    --repository-name <my-public-repo> \
    --region us-east-1
```

### Getting Public ECR Credentials

Authenticate Docker to a public ECR registry. Use the `ecr-public` command, and note that the region **must** be `us-east-1`:

```bash
aws ecr-public get-login-password --region us-east-1 --profile <user-profile> \
  | docker login --username AWS --password-stdin \
    public.ecr.aws
```

Replace `<user-profile>` with your profile. The login host for the public gallery is always `public.ecr.aws`; including the registry alias is unnecessary. A successful login prints `Login Succeeded`.

> **Note:** The token returned by `get-login-password` is valid for **12 hours**. After it expires, re-run this step. The public login endpoint only responds in `us-east-1`.

### Tag and Push the Image

To push an image to public ECR, tag the image, replacing `<registry_alias>` with the repository name. This tells ECR where the image is stored.

```bash
# Tag your local image with the full public ECR URI
docker tag my_geolab:latest \
  public.ecr.aws/<registry_alias>/my_geolab:latest
```

The next step is to push the image to public ECR.

```bash
# Push it
docker push public.ecr.aws/<registry_alias>/my_geolab:latest
```

### Troubleshooting ECR

A 401 Unauthorized or 403 Forbidden are the two most common errors when pushing an image to ECR.

#### 401 Unauthorized

You are **not authenticated**: the Docker auth token is stale, expired, or was never issued.

1. **Refresh your SSO session**, since it likely expired:

   ```bash
   aws sso login --profile <user-profile>
   ```

2. **Mint a fresh Docker token** by re-running the `get-login-password | docker login` step in the "Getting Public ECR Credentials" section.

3. **Confirm your identity** resolves:

   ```bash
   aws sts get-caller-identity --profile <user-profile>
   ```

4. **Check the login region**: `aws ecr-public get-login-password` only works with `--region us-east-1`, and using any other region is a common cause of failed logins.

> A mid-session 401 almost always just means the 12-hour Docker token has expired. Re-login and retry.

#### 403 Forbidden

You **are authenticated**, but you lack permission, or a policy is blocking the action.

1. **Verify IAM permissions.** Your role needs, at minimum:

   - `ecr-public:GetAuthorizationToken`
   - `ecr-public:BatchCheckLayerAvailability`
   - `ecr-public:InitiateLayerUpload`
   - `ecr-public:UploadLayerPart`
   - `ecr-public:CompleteLayerUpload`
   - `ecr-public:PutImage`

2. **Confirm the repository exists.** Pushing to a nonexistent repo returns **403**, not 404:

   ```bash
   aws ecr-public describe-repositories --region us-east-1 --profile <user-profile>

   # If missing, create it:
   aws ecr-public create-repository --repository-name myrepo --region us-east-1 --profile <user-profile>
   ```

3. **Check the repository policy and registry permissions**: a resource-based policy may be explicitly denying your role.

4. **Make sure you assumed the right account and role.** Using a profile that lacks push rights is the most common cause.

#### Clearing credentials and starting fresh

If refreshing your session doesn't fix a 401/403, stale cached credentials are a common culprit. Clear both AWS and Docker credentials, then log in again from scratch.

**Clear AWS credentials:**

Clear active session variables.

```bash
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

Clear cached roles.

```bash
# Clear cached CLI credentials
rm -rf ~/.aws/cli/cache
```

Clear Docker ECR credentials. This removes the entry from Docker's credential store (`~/.docker/config.json`).

```bash
# Log out of the ECR Public registry
docker logout public.ecr.aws
```

After clearing all credentials, re-run the full login sequence from the "Getting Public ECR Credentials" step:

```bash
aws ecr-public get-login-password --region us-east-1 --profile <user-profile> \
  | docker login --username AWS --password-stdin public.ecr.aws
```

## Quick reference

| Error | Meaning | First thing to try |
|-------|---------|--------------------|
| `401 Unauthorized` | Not authenticated / token expired | `aws sso login` then re-run `docker login` |
| `403 Forbidden` | Authenticated but not permitted | Check IAM permissions and that the repo exists |

## Token lifetimes at a glance

- **SSO session:** ~8 to 12 hours (configurable by your admin)
- **ECR Docker token:** 12 hours

When either expires, re-authenticate from the relevant step above.
