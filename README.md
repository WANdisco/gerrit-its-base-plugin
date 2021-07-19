# WANdisco replicated its-base plugin for Gerrit Code Review

This plugin provides the core functionality for interacting with Issue Tracking
Systems, for example: Jira, Redmine or Github Issues.

The WANdisco version of this plugin allows installation to multiple replicated
GerritMS sites where each site will perform validation or enforcement of a
ticket reference in commit messages and a single site made responsible for
interacting with the ITS, i.e. Posting comments or updating ticket status based
on gerrit event triggers.

## Install an ITS plugin

This its-base plugin should have been used as a dependency to build one of the
supported ITS plugins, e.g. its-jira, its-github, its-redmine or
its-phabricator. In this document, we'll use its-jira as the example.

Once you have installed GerritMS on all nodes, to install the its-* plugin, copy
the plugin jar into the plugins folder on all sites you wish the plugin to
run. We recommend installing to all sites:

``` cp -piv <path-to>/its-jira.jar <GERRIT-SITE>/plugins/```

### Configuring the plugin

Configuration is done at the gerrit site level and also at project.config level.

To configure the connection and authentication to a jira server, add the
following section to <GERRIT-SITE>/etc/secure.config:

    [its-jira]
        url = http://jira.example.com
        username = jira.user
        password = !jiraPassword123

To configure the ticket association policy and comment links, add the following
sections to <GERRIT-SITE>/etc/gerrit.config.  Explanations of the options will
follow:

   [plugin "its-jira"]
       association = SUGGESTED
       serverRole = VALIDATE_AND_POST
   [commentlink "its-jira"]
       match = ([A-Z]+-[0-9]+)
       html = "<href a=\"http://jira.example.com/browse/$1\">$1</a>"

1. "association" tells the plugin how it should enforce commit messages having a
   ticket ID. The value is one of:

   * OPTIONAL A ticket ID is not required
   * SUGGESTED A a ticket ID is not supplied, Gerrit will return a hint to the
               user's console that it's recommended to supply one in the
               message.
   * MANDATORY A ticket ID is required, the push will fail if any new commits
               have not supplied one.

2. "serverRole" tells the plugin how it should act on gerrit event triggers
    defined in the actions file. The value is one of:

    * VALIDATE_AND_POST The server will validate commit messages according to
                        the association level, and also act on event triggers.
    * VALIDATE_ONLY The server will only validate commit messages.

3. "match" defines the regular expression used to identify a ticket ID in the
   commit message.

4. "html" defines the link format that will be substituted in the UI so that
    user's can click to an ticket directly from a gerrit review.

Note: When using an its-plugin built with the WANdisco modified its-base, we
recommend having a single server configured with the VALIDATE_AND_POST server
role and all other sites configured to VALIDATE_ONLY to avoid multiple sites
posting to the ITS. We provide the configuration entry so that roles can be
changed easily in the event that a site goes down.

Now restart Gerrit to complete plugin installation.

### Configuring a project

To link a gerrit project to an ITS project, modify the project.config either by
checking out refs/meta/config or via the Gerrit Project commands and add a
section to enable the plugin, name the its-project and optionally define a
branch pattern, e.g:

    [plugin "its-jira"]
        enabled = true
        its-project = project_name
        branch = refs/heads/master
