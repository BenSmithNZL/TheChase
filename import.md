---
editor_options: 
  markdown: 
    wrap: 72
title: "Import"
---

## Import

We will need the readxl package to read in the data, and GenderInfer to
assign a gender to the players.

``` r
library(readxl)
library(GenderInfer)
data <- read_excel("Documents/ChaseData.xlsx")
```

Lets take a peek at the data.

``` r
head(data)
```

And a summary of it

``` r
summary(r)
```

## Cleaning

### Player position

We'll take the "P" out of the player positions, and convert it to a
factor.

``` r
data$Player <- as.factor(gsub("P", "", data$Player))
```

### Cash builder

Taking the pound sign out of the cash builder column.

``` r
data$CashBuilder <- gsub("£", "", data$CashBuilder)
data$CashBuilder <- as.numeric(gsub(",", "", data$CashBuilder))
```

### Low offer

Taking the pound sign out of the low offer column. There's some rows
with a low offer of less than a pound, so we'll convert those to a
proper numeral. There's also some times where no low offer was made, so
we'll have to convert them to NA. Luckily, when we use as.numeric(),
it'll do that for us automatically.

``` r
data$Low <- gsub("£", "", data$Low) 
data$Low[which(data$Low == "1p")] <- 0.01 
data$Low[which(data$Low == "2p")] <- 0.02 
data$Low <- as.numeric(gsub(",", "", data$Low))
```

### High offer

Taking the pound sign out of the high offer column and converting to
numeric.

``` r
data$High <- gsub("£", "", data$High)
data$High <- as.numeric(gsub(",", "", data$High))
```

### Chosen

Taking the pound sign and all of the punctuation out of the offer
selected column.

``` r
data$Chosen <- gsub("£", "", data$Chosen) data$Chosen <- gsub("\\/", "", data$Chosen) data$Chosen <- gsub("\\\\", "", data$Chosen) data$Chosen <- gsub("=", "", data$Chosen) data$Chosen <- as.numeric(gsub(",", "", data$Chosen))
```

### Head-to-head result

Lets make a dummy variable showing if the contestant won their head to
head or not

``` r
data$HTHResult[grep("Home", data$HTHResult)] <- 1 
data$HTHResult[grep("Caught", data$HTHResult)] <- 0 
data$HTHResult <- as.numeric(data$HTHResult)
```

### Final chase correct answers

How many correct answers the player got in the final chase.

``` r
data$FCCorAns <- gsub(" .*", "", data$FCCorAns) 
data$FCCorAns <- as.numeric(data$FCCorAns)
```

### Final chase result

How many correct answers the player got in the final chase.

``` r
data$FCWinner[grep("Team", data$FCWinner)] <- 1 
data$FCWinner[grep("Chaser", data$FCWinner)] <- 0  
data$FCWinner <- as.numeric(data$FCWinner)
```

### Amount won

Taking the pound sign out of the amount won column.

``` r
data$AmountWon <- gsub("£", "", data$AmountWon) 
data$AmountWon <- as.numeric(gsub(",", "", data$AmountWon))
```

### Check

Lets check out our data again.

``` r
head(data) 
summary(data)
```

That looks much better now.

## Adding features

Now that we have a nice, clean dataset, lets start adding some features
that we might need later on.

### Player ID

We can assign each player a unique number.

``` r
data$PlayerID <- as.factor(seq(1, nrow(data), 1))
```

### Team ID

We can assign each team a unique number.

``` r
data$TeamID <- NA

data$TeamID[seq(1, nrow(data), 4)] <- seq(1, nrow(data) / 4, 1)
data$TeamID[seq(2, nrow(data), 4)] <- seq(1, nrow(data) / 4, 1)
data$TeamID[seq(3, nrow(data), 4)] <- seq(1, nrow(data) / 4, 1)
data$TeamID[seq(4, nrow(data), 4)] <- seq(1, nrow(data) / 4, 1)
```

### Gender

``` r
gender <- data.frame(Name = data$Name,
                     ID = data$PlayerID)

gender <- assign_gender(data_df = gender, "Name")

data$Male <- gender[order(gender$ID), 2]

data$Male[which(data$Male == "M")] <- 1
data$Male[which(data$Male == "F")] <- 0
data$Male[which(data$Male == "U")] <- NA
```

### Offer chosen

Which offer is chosen as a factor

``` r
data$OfferChosen <- NA

data$OfferChosen [which(data$Chosen == data$Low)] <- "Low"
data$OfferChosen [which(data$Chosen == data$CashBuilder)] <- "Middle"
data$OfferChosen [which(data$Chosen == data$High)] <- "High"
data$OfferChosen  <- as.factor(data$OfferChosen)
```

### Opportunity cost, risk premium, spread

``` r
data$OC <- abs(data$CashBuilder - data$Low)
data$RP <- abs(data$CashBuilder - data$High)
data$Spread <- abs(data$High - data$Low)
```

### Amount banked

Money in the bank when the player makes their choice

``` r
data$Banked <- NA
data$Banked[seq(1, nrow(data), 4)] <- 0
data$Banked[seq(2, nrow(data), 4)] <- data$Chosen[seq(1, nrow(data), 4)] * data$HTHResult[seq(1, nrow(data), 4)]
data$Banked[seq(3, nrow(data), 4)] <- data$Chosen[seq(2, nrow(data), 4)] * data$HTHResult[seq(2, nrow(data), 4)] + data$Banked[seq(2, nrow(data), 4)]
data$Banked[seq(4, nrow(data), 4)] <- data$Chosen[seq(3, nrow(data), 4)] * data$HTHResult[seq(3, nrow(data), 4)] + data$Banked[seq(3, nrow(data), 4)]
```

### Players home

Amount of players home when the player makes their choice

``` r
data$Home <- NA

data$Home[seq(1, nrow(data), 4)] <- 0
data$Home[seq(2, nrow(data), 4)] <- data$HTHResult[seq(1, nrow(data), 4)]
data$Home[seq(3, nrow(data), 4)] <- data$Home[seq(2, nrow(data), 4)] + data$HTHResult[seq(2, nrow(data), 4)]
data$Home[seq(4, nrow(data), 4)] <- data$Home[seq(3, nrow(data), 4)] + data$HTHResult[seq(3, nrow(data), 4)]
```

All done! Now we can start exploring the data.
