# Nextra Vercel Example

This example repository shows how to publish Modify managed content to a Nextra based static site
using Github Actions and Vercel.

## Nextra

Nextra is a Next.js based static site generator. The root of this repository contains a simple setup
using the docs theme, and is intended to be built and deployed to Vercel.

To test it locally you need NodeJS. We recommend using `nvm` for Node, setup instructions are
available at https://github.com/nvm-sh/nvm. 

Install correct Node version (if using nvm)
```bash
nvm install
```

Install packages and serve locally
```bash
npm install
npm run start
```
Site should be accessible on http://localhost:3000/

## `update_pages.sh`

This script is designed to be run by Github Actions and will
- Download markdown content from a Modify connector
- Empty `pages` and unpack the downloaded content to replace it
- Commit and push changes to Github
- Notify Modify it has completed (if provided with a JOB_INSTANCE_ID)

## Github Actions

There is a Github Actions workflow defined in `.github/workflows/main.yml` which will be run by
Modify Jobs. It can also be run manually from Github Actions UI provided the required `inputs` are
passed:

- `refresh_token`: Modify refresh token
- `team_slug`: Team slug
- `workspace_slug`: Workspace slug
- `workspace_branch_slug`: Workspace branch slug (defaults to `master`)
- `connector_slug`: Connector slug (defaults to `pages`)
- `connector_path_slug`: Connector path (defaults to `/`)

## Vercel

Vercel will be used to build and host the static site: https://vercel.com/

## Step 1 - Github configuration

The publication cycle of this setup requires changes to be committed and pushed to Github in order
for Vercel to use them, so you need to fork the repository e.g. `my-org/nextra-vercel-example`.

Once forked, the workflow will be disabled by default. To enable it, go to `Actions` in the Github
console and click the `I understand my workflows, go ahead and enable them` button.

Finally, you will need to generate a Github Personal Access Token to allow Modify to trigger the
workflow. You can do this at https://github.com/settings/tokens. It requires full control of private
repositories as changes need to be committed in order to update posts.

## Step 2 - Vercel configuration

If you do not already have a Vercel account, you can sign up here https://vercel.com/signup. You
will need to authorise Vercel to access your Github account in the steps below if this is a new
account.

Go to the Vercel dashboard and click `Import Project` and `Import Git Repository`. Then enter the
URL of your github repository (https://github.com/my-org/nextra-vercel-example) and click continue.
It should detect that you have a `Next.js` project and give it a default name of
`nextra-vercel-example`. Click `Deploy` to build and deploy.

Once the site is built, you can view it by clicking on the `Visit` button. The URL will be unique
to your deployment, but the default is https://nextra-vercel-example.vercel.app/

## Step 3 - Setup Modify

The following steps assume you have a Modify team with the slug `my-team`.

Create a new workspace with the slug `nextra-vercel-example` and base branch ID `master`.

Create a new Modify connector in your workspace called `pages` with `Editable` access mode.

Add a file to the connector called `/index.md` with the following content:
```
# Home

This is my home page
``` 
and commit your changes.

If you have an existing connector that you would like to use, then you will need to adjust the Job
Definition payload in [Step 4](#step-4---create-modify-job) to suit.

## Step 4 - Create Modify Job

In Modify, select the correct team and workspace and go to the Jobs section.

Click the `Create Job` button and then select the `Example: Publish Nextra to Vercel` template:

You will need to complete the following fields:

- Github Owner: `my-org`
- Github Repository: `nextra-vercel-example`

Next click `+` next to Credentials and enter the following:
- Name: `Nextra Vercel Example`
- Username: `<Github username>`
- Password: `<Github personal access token>`

Then click `Add Credential` to create the new credential and return to the `Create Job` form.

Your new credential should have been selected automatically, so you can finally click `Create` to
save your new Job Definition.

Click the name of the new Job Definition to display the details.

## Step 5 - Run the Modify Job

Click `Start` to run the Job and a new Job Instance will be created. This will `POST` the payload to
the Github API gateway along with configured credentials, `REFRESH_TOKEN` and `JOB_INSTANCE_ID`.
Modify expects the remote system to notify when the job is complete using these details.

When this is complete you should see the Job Instance in Modify change from `Started` to `Finished`.
Vercel will be notified that the Github repository has been updated and will build and deploy the
site. This shouldn't take more than a few minutes and it's progress can be viewed in the Vercel
control panel.
