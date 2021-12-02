# Github Api

A simple example to understand how to use github api.

```
/* Just execute the ':. ! bash' in Vim command mode. The cursor must be on the command below. It's better to backup the command before doing it.
curl --silent https://api.github.com | sed  -E 's/",/",\n/'

/* Get all query url.

{
  "current_user_url": "https://api.github.com/user",

  "current_user_authorizations_html_url": "https://github.com/settings/connections/applications{/client_id}",

  "authorizations_url": "https://api.github.com/authorizations",

  "code_search_url": "https://api.github.com/search/code?q={query}{&page,per_page,sort,order}",

  "commit_search_url": "https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}",

  "emails_url": "https://api.github.com/user/emails",

  "emojis_url": "https://api.github.com/emojis",

  "events_url": "https://api.github.com/events",

  "feeds_url": "https://api.github.com/feeds",

  "followers_url": "https://api.github.com/user/followers",

  "following_url": "https://api.github.com/user/following{/target}",

  "gists_url": "https://api.github.com/gists{/gist_id}",

  "hub_url": "https://api.github.com/hub",

  "issue_search_url": "https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}",

  "issues_url": "https://api.github.com/issues",

  "keys_url": "https://api.github.com/user/keys",

  "label_search_url": "https://api.github.com/search/labels?q={query}&repository_id={repository_id}{&page,per_page}",

  "notifications_url": "https://api.github.com/notifications",

  "organization_url": "https://api.github.com/orgs/{org}",

  "organization_repositories_url": "https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}",

  "organization_teams_url": "https://api.github.com/orgs/{org}/teams",

  "public_gists_url": "https://api.github.com/gists/public",

  "rate_limit_url": "https://api.github.com/rate_limit",

  "repository_url": "https://api.github.com/repos/{owner}/{repo}",

  "repository_search_url": "https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}",

  "current_user_repositories_url": "https://api.github.com/user/repos{?type,page,per_page,sort}",

  "starred_url": "https://api.github.com/user/starred{/owner}{/repo}",

  "starred_gists_url": "https://api.github.com/gists/starred",

  "topic_search_url": "https://api.github.com/search/topics?q={query}{&page,per_page}",

  "user_url": "https://api.github.com/users/{user}",

  "user_organizations_url": "https://api.github.com/user/orgs",

  "user_repositories_url": "https://api.github.com/users/{user}/repos{?type,page,per_page,sort}",

  "user_search_url": "https://api.github.com/search/users?q={query}{&page,per_page,sort,order}"
}

/* Then try to search a repo.

curl --silent https://api.github.com/search/repositories?q=exfat | grep release

      "releases_url": "https://api.github.com/repos/relan/exfat/releases{/id}",

/* Then try to get the exfat latest url.

curl --silent https://api.github.com/repos/relan/exfat/releases/latest

      "browser_download_url": "https://github.com/relan/exfat/releases/download/v1.3.0/exfat-utils-1.3.0.tar.gz"
      "browser_download_url": "https://github.com/relan/exfat/releases/download/v1.3.0/fuse-exfat-1.3.0.tar.gz"

/* Finally download all the release packages.

$ curl -L -O https://github.com/relan/exfat/releases/download/v1.3.0/exfat-utils-1.3.0.tar.gz

$ curl -L -O https://github.com/relan/exfat/releases/download/v1.3.0/fuse-exfat-1.3.0.tar.gz

```

------
