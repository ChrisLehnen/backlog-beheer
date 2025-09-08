# Dashboard Directory

This directory will contain the HTML dashboard and related assets for backlog visualization.

## Structure
```
dashboard/
├── index.html          # Main dashboard page
├── graph.html          # Relationship visualization
├── data.json           # Generated backlog data
└── assets/
    ├── css/           # Stylesheets
    ├── js/            # JavaScript files
    └── img/           # Images and icons
```

## Usage
1. Run the data generator script to create `data.json`
2. Open `index.html` in a web browser
3. Use search and filters to navigate the backlog

## Data Generation
```bash
python scripts/generate_dashboard_data.py
```

This will scan all markdown files and generate the JSON data for the dashboard.