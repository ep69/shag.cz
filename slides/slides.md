---
marp: true
---

#  GitHub == free webhosting
##  simple case study

---
<style scoped>section { font-size: 40px; }</style>

## Requirements

- simple static website for my dancing community
- show upcoming events from public Google calendar

---
<style scoped>section { font-size: 40px; }</style>

## Components

- [GitHub Pages](https://pages.github.com/)
- own domain: [shag.cz](shag.cz)
- python with [googleapiclient](https://github.com/googleapis/google-api-python-client) and [jinja2](https://github.com/pallets/jinja/)
- [GitHub Actions](https://docs.github.com/en/actions)

---
<style scoped>section { font-size: 40px; }</style>

## GitHub Pages
- static content from branch, or
- generate dynamically with GitHub Actions
- result: `https://<user>.github.io/<repo>`

---
<style scoped>section { font-size: 40px; }</style>

## Custom domain
- configure DNS A records
- configure GitHub Pages to use it

---
<style scoped>section { font-size: 40px; }</style>

## Fetch events

```python
api_key = os.environ['GAPI_KEY'] # secret managed by GitHub
calendar_id = "<...>@group.calendar.google.com"
maxResults = 10
service = build("calendar", "v3", developerKey=api_key)
events_result = service.events().list(
    calendarId=calendar_id,
    timeMin=now, singleEvents=True, orderBy="startTime",
    maxResults=maxResults, timeZone="Europe/Prague").execute()
```
---
<style scoped>section { font-size: 40px; }</style>

## Parse events
```json
"events": [
    {
        "summary": "Shag open trénink - 4.p.",
        "when": "2024-08-08 17:30 - 19:00"
    },
    ...
]
```

---
<style scoped>section { font-size: 40px; }</style>

## Page template

```html
<h2>Upcoming events:</h2>
<ul>
{% for event in events %}
    <li>
        <span class="summary">{{ event.summary }}</span>
        <span class="when">({{ event.when }})</span>
    </li>
{% endfor %}
```

---
<style scoped>section { font-size: 40px; }</style>
## Generate page

```python
environment = Environment(loader=FileSystemLoader("./"))
page_template = environment.get_template("template.html")
context = {
    "events": events,
    }
with open("index.html", mode="w", encoding="utf-8") as page:
    page.write(page_template.render(context))

```

---
<style scoped>section { font-size: 40px; }</style>
## Publish site

```yaml
on:
  push: # refresh in case of changes
    branches: ["master"]
  schedule: # refresh every hour
    - cron:  '42 * * * *'
  workflow_dispatch: # refresh manually
...
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v5
      - name: Generate index.html
        env:
          GAPI_KEY: ${{ secrets.GAPI_KEY }}
        run: |
          sudo apt-get install -y python3 python3-jinja2 python3-googleapi
          python3 fetch-events.py
          rm -f fetch-events.py
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---
<style scoped>section { font-size: 40px; }</style>
<style>
.container { display: flex; }
.item1 { flex: 1; }
.item2 { flex: 3; }
</style>
<div class="container">
<h2 class="item1">Result</h2>
<img class="item2" src="./web.webp" style="object-fit: contain; max-height: 600px;"/>
</div>

---
<style scoped>section { font-size: 40px; }</style>

## Reference

- [sources](https://github.com/ep69/shag.cz)
- [shag.cz web](https://shag.cz)

