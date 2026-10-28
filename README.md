# Querio workspace template

Starter context for `querio init`. This repository owns the local starter files and context selection defaults.

After initializing:

1. Run `querio login`.
2. Create a hosted workspace with `querio workspace create --name Acme`, or list existing workspaces with `querio workspace list`.
3. Run `querio workspace link <workspace-id>` to bind `querio.toml` to the workspace.
4. Review the context selection in `querio.toml`, then commit the files in your own repository.

For Git snapshot uploads, set `[context].repository` to your stable repository identity or pass `git upload --repository <identity>`.

Do not commit credentials. Review selected files before uploading.
