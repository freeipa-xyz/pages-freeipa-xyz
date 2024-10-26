---
layout: theme
title:  "Password generation in bash"
date:   2024-11-02
theme:  password-generation-bash
permalink: /t/password-generation-bash
---

To generate passwords using `bash`, use this lines:

```
# Strong enough
< /dev/urandom tr -cd "A-Za-z0-9" | head -c 20; echo

# Extremely strong
< /dev/urandom tr -cd "[:graph:]" | head -c 20; echo
```

