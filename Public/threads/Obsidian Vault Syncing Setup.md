This vault contains a mix of public and private files. All files are private except for files located in `./Public` which get copied to a public repo.

### GitHub Workflow (sync public files)

On sync/push, [.github/workflows/sync-public.yaml](./.github/workflows/sync-public.yaml) handles copying public files.

```yml
name: "Sync public files"
on:
	push:
		branches: [ main ]
jobs:
	sync-public:
		name: Sync public files
		runs-on: ubuntu-latest
		steps:
			- name: Sync public files
			uses: nkoppel/push-files-to-another-repository@v1.1.4
			env:
			API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_GITHUB }}
			with:
				source-files: 'Public/'
				destination-username: 'your-username'
				destination-repository: 'destination-repo-name'
				destination-branch: 'main'
				commit-email: 'youremail@email.com'
```
#### Generate GitHub API Key

The default API key only has access to the private repo, so you'll need to generate a new key that has read/write access to both the public and private repos.

- Go to https://github.com/settings/personal-access-tokens/new or navigate to it through GitHub Settings > Developer Settings > Fine-grained tokens > - Generate new token

- Fill out owner, name, and expiration date.

- Set "Repository Access" to "Only select repositories", and select the repositories you would like this action to be able to edit. Alternatively, select "All Repositories" to give it access to all of your repositories, at the cost of security.

- Click into "Repository Permissions" and set "Contents" to "Read and Write"

- Generate and copy the token.

- Go to the GitHub page for the repository that you push from and click into "Settings"

- On the left sidebar, click into Secrets and Variables > Actions

- Click on "New Repository Secrets", name it "API_TOKEN_GITHUB", and paste your token.