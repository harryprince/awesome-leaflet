* convert trip network dataframe to sf and viz

```
t <- read_delim("create_time	|start_positionx|	start_positiony|	end_positionx	|end_positiony
2019-04-02 18:29:56	|114.4975	|36.65130	|114.4939	|36.65469
2019-04-02 09:10:46	|114.4942	|36.65450	|114.4975	|36.65141
2019-04-03 08:38:20	|114.4940	|36.65458	|114.4974	|36.65142
2019-04-02 14:58:54	|114.4935	|36.65602	|114.4974	|36.65137","|")
```

```
cent <- rbind(t %>% select(lng = start_positionx,lat = start_positiony),
      t %>% select(lng = end_positionx,lat = end_positiony)) %>% 
    distinct() %>% na.omit() %>% mutate(gh = geohash::gh_encode(lat,lng,precision = 10)) %>%
    sf::st_as_sf(coords=c("lng","lat"))

flow <- t %>% 
    transmute(src_gh = geohash::gh_encode(start_positiony,start_positionx,10),
              dst_gh = geohash::gh_encode(end_positiony,end_positionx,10))

travel_network <- stplanr::od2line(flow = flow, zones = cent) %>%  
                  sf::st_as_sf() 


t %>%
    arrange(create_time) %>% 
    leaflet::leaflet() %>% 
    leafletCN::amap() %>% 
    leaflet::addCircleMarkers(lng = ~start_positionx,
                              lat = ~start_positiony,
                              color = "green") %>% 
    leaflet::addCircleMarkers(lng = ~end_positionx,
                              lat = ~end_positiony,
                              color = "red") %>% 
    leaflet::addPolylines(data=travel_network)
    ```
