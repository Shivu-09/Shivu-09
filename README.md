name: Profile Refresh

on:
  schedule:
    # Runs every day at 00:30 UTC — after the snake workflow, before the others
    - cron: "30 0 * * *"
  workflow_dispatch: {}

jobs:
  refresh:
    runs-on: ubuntu-latest
    steps:
      - name: Warm up dynamic widget caches
        run: |
          echo "Pinging dynamic README widgets to refresh their cache..."
          curl -s -o /dev/null "https://github-readme-stats.vercel.app/api?username=${{ github.repository_owner }}&show_icons=true" || true
          curl -s -o /dev/null "https://github-readme-streak-stats.herokuapp.com/?user=${{ github.repository_owner }}" || true
          curl -s -o /dev/null "https://github-readme-activity-graph.vercel.app/graph?username=${{ github.repository_owner }}" || true
          curl -s -o /dev/null "https://github-profile-trophy.vercel.app/?username=${{ github.repository_owner }}" || true
          echo "Cache warm-up complete."

  trigger-dependent-workflows:
    needs: refresh
    runs-on: ubuntu-latest
    permissions:
      actions: write
    steps:
      - name: Trigger Snake Animation workflow
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.actions.createWorkflowDispatch({
              owner: context.repo.owner,
              repo: context.repo.repo,
              workflow_id: 'snake.yml',
              ref: 'main'
            });

      - name: Trigger Daily README Update workflow
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.actions.createWorkflowDispatch({
              owner: context.repo.owner,
              repo: context.repo.repo,
              workflow_id: 'daily-readme-update.yml',
              ref: 'main'
            });

      - name: Trigger Quote Update workflow
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.actions.createWorkflowDispatch({
              owner: context.repo.owner,
              repo: context.repo.repo,
              workflow_id: 'quote-update.yml',
              ref: 'main'
            });

      - name: Trigger Metrics Update workflow
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.actions.createWorkflowDispatch({
              owner: context.repo.owner,
              repo: context.repo.repo,
              workflow_id: 'metrics.yml',
              ref: 'main'
            });
