---
created: 2024-03-26T14:10
updated: 2024-03-26T14:23
---
find . -type f -name "*.md" -exec sed -i 's/```apacheconf/```apache/g' {} +

or rsync to sync? 