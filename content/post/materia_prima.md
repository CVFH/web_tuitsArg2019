---
date: "2020-06-17T22:01:14-05:00"
tags:
- issue
title: Multiple Expandable Test
enableEmoki: true
---

Testing out GitHub issue https://github.com/zwbetz-gh/cupper-hugo-theme/issues/36 -- Multiple expandable shortcodes do not work if they have the same inner text.



{{< expandable label="Funciones" level="2" >}}
Same inner text.
{{< /expandable >}}

```{r cool-plot, fig.cap='A cool plot.'}
{{<code numbered="true">}}
[[[plot(cars, pch = 20)]]]  # not really cool
{{</code>}}
aver <- "value" # asi asignamos valores
```


```{r cool-plot, fig.cap='A cool plot.'}
plot(cars, pch = 20)  # not really cool
```


{{< expandable label="Preparando Datos" level="2" >}}
Same inner text.
{{< /expandable >}}
