Project 1
================

# NHL Data statistics

## Functions

This program provides functions to access NHL records and stats data

### get\_franchise()

this function returns id, firstSeasonId and lastSeasonId and name of
every team in the history of the NHL

``` r
#Usage
franchiseData <- get_franchise()
```

### get\_team\_totals()

This function returns Total stats for every franchise (ex roadTies,
roadWins, etc)

``` r
#Usage
teamTotals <- get_team_totals()
```

### get\_season\_records(ID,name)

This function drill-down into season records for a specific franchise.
Caller can provide team ID or name.

``` r
#Usage
seasonRecords <- get_season_records(ID=1)
seasonRecords <- get_season_records(name="team name")
```

### get\_goalie\_records(ID,name)

This function drill-down into goalie records for a specific franchise.
Caller can provide team ID or name.

``` r
#Usage
goalieRecords <- get_goalie_records(ID=1)
goalieRecords <- get_goalie_records(name="team name")
```

### get\_skater\_records(ID,name)

This function drill-down into skater records for a specific franchise.
Caller can provide team ID or name.

``` r
#Usage
skaterRecords <- get_skater_records(ID=1)
skaterRecords <- get_skater_records(name="team name")
```

### get\_franchise\_details(ID)

This function drill-down into franchise details by records for a
specific franchise. Caller can provide most recent Team ID

``` r
#Usage
franchiseDetails <- get_franchise_details(ID=1)
```

### get\_team\_stats(teamID)

This function provides team stats. If ID is provided, it will return the
data for specific team, otherwise it will send for all the teams.

``` r
#Usage
teamStats <- get_team_stats(teamID=1)
```

### get\_nhl\_data(endpoint, id,name)

Caller can use this wrapper function to get corresponding data.This is
one-stop-shop function for caller to use

``` r
#Usage
franchiseData <- get_nhl_data("Franchise")
skaterRecords <- get_nhl_data("SkaterRecords", id= 2, name="team name")
```

## Using Functions

We will use above functions to create some data tables and plots

### Combining two tables

Below code uses two functions to get the data and combines them into one
data table

``` r
franchiseData <- get_franchise() %>% rename (franchiseId = id )
combinedFranchiseData <- left_join(franchiseData, get_team_totals(), by=c("franchiseId","lastSeasonId"))

knitr::kable(head(combinedFranchiseData%>% 
          select(franchiseId,fullName,gamesPlayed,gameTypeId, wins,losses,ties,overtimeLosses)))
```

| franchiseId | fullName            | gamesPlayed | gameTypeId | wins | losses | ties | overtimeLosses |
|------------:|:--------------------|------------:|-----------:|-----:|-------:|-----:|---------------:|
|           1 | Montréal Canadiens  |         773 |          3 |  444 |    321 |    8 |              0 |
|           1 | Montréal Canadiens  |        6787 |          2 | 3473 |   2302 |  837 |            175 |
|           2 | Montreal Wanderers  |           6 |          2 |    1 |      5 |    0 |             NA |
|           3 | St. Louis Eagles    |          48 |          2 |   11 |     31 |    6 |             NA |
|           4 | Hamilton Tigers     |         126 |          2 |   47 |     78 |    1 |             NA |
|           5 | Toronto Maple Leafs |        6516 |          2 | 2873 |   2696 |  773 |            174 |

### Adding variables

We will add two new columns to the data that we have combined as
WinLossRatio and Home Win percentage

``` r
# select few columns rather than using all 37 columns
filteredData <- combinedFranchiseData %>% filter(activeFranchise ==1, !is.null(lastSeasonId)) %>% select(franchiseId,fullName,gamesPlayed,gameTypeId, wins,losses,ties,overtimeLosses, homeWins,homeLosses, homeTies,homeOvertimeLosses) 

# add two new columns WinLoss ratio and Home Win Percentage
customData2 <- filteredData %>% mutate(winLossRatio = wins/losses, homeWinPctg = (homeWins/gamesPlayed)*100)

#Adding one more column to create some groups
customData3 <- customData2 %>% mutate(homeWinPctgGroup = case_when(
    homeWinPctg > 50 ~ 50,
    homeWinPctg > 40 ~ 40,
    homeWinPctg > 35 ~ 35,
    homeWinPctg > 30 ~ 30,
    homeWinPctg > 25 ~ 25,
    homeWinPctg > 20 ~ 20,
    homeWinPctg > 15 ~ 15,
    homeWinPctg > 10 ~ 10,
    homeWinPctg < 10 ~ 5,
    ))
knitr::kable(head(customData3%>% 
          select(franchiseId,fullName,gamesPlayed,winLossRatio,homeWinPctg,homeWinPctgGroup ),10))
```

| franchiseId | fullName            | gamesPlayed | winLossRatio | homeWinPctg | homeWinPctgGroup |
|------------:|:--------------------|------------:|-------------:|------------:|-----------------:|
|           1 | Montréal Canadiens  |         773 |    1.3831776 |    33.37646 |               30 |
|           1 | Montréal Canadiens  |        6787 |    1.5086881 |    30.02799 |               30 |
|           5 | Toronto Maple Leafs |        6516 |    1.0656528 |    26.12032 |               25 |
|           5 | Toronto Maple Leafs |         545 |    0.9151943 |    27.33945 |               25 |
|           6 | Boston Bruins       |        6626 |    1.3487308 |    28.44854 |               25 |
|           6 | Boston Bruins       |         675 |    0.9851632 |    28.74074 |               25 |
|          10 | New York Rangers    |        6560 |    1.0614875 |    24.60366 |               20 |
|          10 | New York Rangers    |         518 |    0.9172932 |    26.44788 |               25 |
|          11 | Chicago Blackhawks  |         548 |    0.9745455 |    30.29197 |               30 |
|          11 | Chicago Blackhawks  |        6560 |    1.0184716 |    25.22866 |               25 |

### Contigency table

Create contingency table using the data that we have just prepared. \*
First one prepares the data for team names \* Second table is for Home
Wins Group with corresponding game types (regular or playoff)

``` r
table(customData3$fullName)
```

    ## 
    ##         Anaheim Ducks       Arizona Coyotes         Boston Bruins 
    ##                     2                     2                     2 
    ##        Buffalo Sabres        Calgary Flames   Carolina Hurricanes 
    ##                     2                     2                     2 
    ##    Chicago Blackhawks    Colorado Avalanche Columbus Blue Jackets 
    ##                     2                     2                     2 
    ##          Dallas Stars     Detroit Red Wings       Edmonton Oilers 
    ##                     2                     2                     2 
    ##      Florida Panthers     Los Angeles Kings        Minnesota Wild 
    ##                     2                     2                     2 
    ##    Montréal Canadiens   Nashville Predators     New Jersey Devils 
    ##                     2                     2                     2 
    ##    New York Islanders      New York Rangers       Ottawa Senators 
    ##                     2                     2                     2 
    ##   Philadelphia Flyers   Pittsburgh Penguins       San Jose Sharks 
    ##                     2                     2                     2 
    ##       St. Louis Blues   Tampa Bay Lightning   Toronto Maple Leafs 
    ##                     2                     2                     2 
    ##     Vancouver Canucks  Vegas Golden Knights   Washington Capitals 
    ##                     2                     2                     2 
    ##         Winnipeg Jets 
    ##                     2

``` r
table(customData3$homeWinPctgGroup,customData3$gameTypeId)
```

    ##     
    ##       2  3
    ##   10  0  1
    ##   15  0  2
    ##   20  8  4
    ##   25 21 14
    ##   30  2 10

### Data Summary

``` r
customData3 %>% group_by(homeWinPctgGroup) %>% summarise(count=n(), Mean_HomeWins = round(mean(homeWins),2))
```

    ## # A tibble: 5 x 3
    ##   homeWinPctgGroup count Mean_HomeWins
    ##              <dbl> <int>         <dbl>
    ## 1               10     1            6 
    ## 2               15     2           11 
    ## 3               20    12          477.
    ## 4               25    35          603.
    ## 5               30    12          268.

## Plots

### Box Plot

For boxplot we will use team names and corresponding losses. We have two
records for each team (one from regular and other from playoffs), we are
using first 5 teams only.

``` r
# Take first 5 teams
selectedData <- customData3 %>% filter(franchiseId < 12)

# Box plot
ggplot(selectedData, mapping = aes(x = fullName, y = losses, fill=fullName)) +
  geom_boxplot(outlier.shape = NA) + 
  ggtitle("Boxplot for losses") +
  theme(legend.position="none") +
    scale_fill_brewer(palette="Dark2")
```

![](/Users/Rashmi/MS%20Work/ST558/Homeworks/NHLstats/README_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

### BarPlot

Below BarPlot uses Home Win Percentage Group

``` r
#Regular game data
regularGame <- customData3 %>% filter(gameTypeId==3)

g <- ggplot(data = regularGame , aes(x = homeWinPctgGroup))
g + geom_bar() + labs(x = "Home Win % by range")
```

![](/Users/Rashmi/MS%20Work/ST558/Homeworks/NHLstats/README_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

### Histogram

Below histogram shows Overtime Losses and frequency.

``` r
ggplot(customData3, aes(x = losses, y = ..density..)) + 
  geom_histogram(bins = 20, size = 0.2) + 
  geom_density(color="red",size=2,outline.type="full") +
   ggtitle("Histogram of Losses")
```

![](/Users/Rashmi/MS%20Work/ST558/Homeworks/NHLstats/README_files/figure-gfm/unnamed-chunk-23-1.png)<!-- -->

``` r
ggplot(customData3, aes(x = wins, y = ..density..)) + 
  geom_histogram(bins = 20, size = 0.2) + 
  geom_density(color="red",size=2,outline.type="full") +
   ggtitle("Histogram of Wins")
```

![](/Users/Rashmi/MS%20Work/ST558/Homeworks/NHLstats/README_files/figure-gfm/unnamed-chunk-23-2.png)<!-- -->

### Scattered plot

Below scattered plot displays homewins against home win percentage and
grouped by homewinsgroup

``` r
ggplot(customData3, aes(x=homeWins, y=homeWinPctg, color=homeWins)) + 
    geom_point() +
  facet_grid(. ~ homeWinPctgGroup) +
  labs(x = "Home Win Percentage Range")
```

![](/Users/Rashmi/MS%20Work/ST558/Homeworks/NHLstats/README_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->
