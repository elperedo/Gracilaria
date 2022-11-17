## Rarefaction curve

```
rarefaction_observed_features <- read.csv("~/evol5/users/eperedo/Gracilaria_Analysis/rarefaction_observed_features.csv")
library(tidyr)
library(reshape2)
library(ggplot2)
library(viridis)
library(data.table)

data <-melt(rarefaction_observed_features, id.vars=c("sample.id", "nutrient", "species"))

data2 <-data[data$variable %like% "iter.2", ] 
data2 %>%
  ggplot( aes(x=variable, y=value, group=sample.id, color=species)) +
  geom_line() +
  scale_color_viridis(discrete = TRUE) +
  theme(
    legend.position="right",
    plot.title = element_text(size=14)
  ) +
  facet_wrap(~nutrient)

```
