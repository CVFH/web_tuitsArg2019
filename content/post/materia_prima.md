---
date: "2020-01-26"
tags:
- preparacion
title: Preparación
enableEmoki: true
---

Testing out GitHub issue https://github.com/zwbetz-gh/cupper-hugo-theme/issues/36 -- Multiple expandable shortcodes do not work if they have the same inner text.



{{< expandable label="Funciones" level="2" >}}
Same inner text.
{{< /expandable >}}


{{<code numbered="true">}}
```{r cool-plot, fig.cap='A cool plot.'}
plot(cars, pch = 20)  [[[# not]]] really cool
```
{{</code>}}

{{< expandable label="Preparando Datos" level="2" >}}
Same inner text.
{{< /expandable >}}
