# Triggering Actions from Projects v2 events

GitHub Actions currently cannot be triggered directly by `projects_v2_item` webhook events. To run workflows when Projects (new) items change, forward the Projects webhook to this repository using a `repository_dispatch` event.

Two common approaches:

- Create a small GitHub App (recommended) subscribed to the `projects_v2_item` webhook for your organization. When the App receives the webhook, it calls the Repositories Dispatch API to forward the payload to the target repo.
- Use an organization webhook + an intermediate forwarder service (self-hosted) that receives `projects_v2_item` and calls the Repositories Dispatch API.

Example: forwarder using `curl` (requires a token with `repo` scope for the target repo):

```
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"projects_v2_item","client_payload": {"project_node_id":"...","item_node_id":"...","action":"created"}}'
```

What to send as `client_payload`:
- `project_node_id` — node id of the project (string)
- `item_node_id` — node id of the item (string)
- `action` — action name (e.g. `created`, `edited`, `reordered`)

The repository workflow `/.github/workflows/projects-dates-dispatch.yml` listens for `repository_dispatch` with `event_type` `projects_v2_item` and will use the values from `client_payload`.

Permissions/notes:
- The token used to call the dispatch endpoint needs `repo` scope for private repos (or `public_repo` for public repos).
- If you use a GitHub App, use the App to authenticate and call the REST API on behalf of the App installation.
- Keep the forwarder (or App) secure — it will need a credential able to dispatch events to your repo.

If you want, můžu připravit ukázkovou GitHub App implementaci (Node.js) nebo jednoduchý forwarder jako Cloud Function — kterou preferujete? 
