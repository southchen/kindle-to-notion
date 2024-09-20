# 🚀 Kindle to Notion 
### Seamlessly transfer your Kindle highlights to a Notion Database!

## See original readme at https://github.com/arkalim/kindle-to-notion

# ⚙️ Setup

- Duplicate my [Notion books database](https://arkalim.notion.site/Library-c966166d851b4a3588bf33049175dd79) as a template in your Notion account. You don't need to clone the contents of my database. You only need to database table. This will serve as your Notion books database.

- Create a new **internal integration** at https://www.notion.so/my-integrations and copy the **Notion API key**.
![](/images/book-highlights-integration.png)

- Go to your Notion dashboard. Open your books database as a page. 
![](/images/open-books-db-as-a-page.png)

- Click on the three dots icon (more options) and go to **Add connection**. Search for the integration you created in the previous step and add it as a connection.
![](/images/adding-integration-to-database.png)

- On the Notion books database page, go to **Share** and click on **Copy link** to get the URL of the Notion database.
**Note:** You must open the books database **as a page** before copying the link.
![](/images/getting-db-link.png)

- Now, extract the Database Id from the link as shown in the example below:
  ```
  Book Database Link: https://www.notion.so/arkalim/346be84507ff482b80fceb4024deadc2?v=e868075eaf5749bc941e617e651295fb
  Book Database Id: 346be84507ff482b80fceb4024deadc2
  ```
- Connect your **Kindle** E-Reader to your computer. Navigate to `Kindle` ➡ `documents` and copy `My Clippings.txt`. 

- Create a GitHub repository and add your `My Clippings.txt` to the root of the repository. Now, add a file `sync.yaml` to the repository at location `.github/workflows` with the following content:
  
```
 name: Sync Kindle Highlights
 on: [push, workflow_dispatch]
 jobs:
  sync-highlights:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Sync highlights
        uses: addnab/docker-run-action@v3
        with:
          image: ghcr.io/southchen/kindle-to-notion:master
          run: node /code/dist/main.js
          options: |
            -v ${{ github.workspace }}:/code/resources
            -e BOOK_DB_ID=${{ vars.BOOK_DB_ID }}
            -e NOTION_API_KEY=${{ secrets.NOTION_API_KEY }}

      - name: Commit cache changes
        uses: EndBug/add-and-commit@v9
        with:
          message: Commit cache changes
          add: sync.json

      - name: Push cache changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
  ```

- The above step will trigger a GitHub action to sync the highlights. But, it will fail as you haven't yet setup the `NOTION_API_KEY` and the `BOOK_DB_ID`. To do that, go to the GitHub repository settings, click on *Secrets and Variables* and then click on *Actions*. Now, create a secret by the name `NOTION_API_KEY` and a variable by the name `BOOK_DB_ID`, with their corresponding values as obtained in the previous steps.
![](/images/configuring-secrets.png)

- Now, you need to allow GitHub Action to save the cache (make updates) to the repository. For that, go to repository settings. Click on *Actions* and then on *General*. Scroll down to *Workflow permissions* and select *Read and write permissions*.
![](/images/workflow-permissions.png)

- You can now manually trigger the GitHub Action by going to the Actions tab in the GitHub repo. This will sync the highlights for the first time and create a `sync.json` file (cache) in the repository. Afterwards, whenever you want to sync your highlights, just upload the new `My Clippings.txt` file into your GitHub repo. It will automatically trigger the action to sync the newly added highlights.


