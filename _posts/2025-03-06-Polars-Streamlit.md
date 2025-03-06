---
layout: post
title: "Updated my Streamlit app to use Polars"
date: 2025-03-6
author: "Amit Indap"
tags:
  - Python
---

I had [posted ealier](https://indapa.github.io/2025/02/17/Polars.html) about exploring [Polars](https://pola.rs/), a Rust based dataframe library with a Python frontend. Polars is similar to Pandas, but is faster and more memory efficient. Polars is also designed to work with large datasets that don't fit into memory.

The best way to learn a new library is to use it in a real project. I have a [Streamlit app](https://indapa-solar.streamlit.app/) that I maintain that displays various data visualizations about my solar panel output. Previously, it was using Pandas to load and manipulate the data. I decided to update the app to use Polars instead.

For example, in my [function to display monthly data](https://github.com/indapa/SolarProduction/blob/master/pages/Comparative_Production.py#L177), I load use a Polars LazyFrame, which means the data isn't loaded into memory until it is needed. Polars optimizes the operations to calculate monthly output and pass the data to Plotly for visualization.

```
def plotly_quarterly_comparative_production():
    
    q = (
    pl.scan_csv(solar_file)
    .with_columns(
            # Convert the Time column to a proper date
            pl.col("Time")
            .str.strptime(pl.Date, format="%Y-%m-%d")
            .alias("Date"),
            
    
        )
        .sort("Date")
        .with_columns (
            
            month = pl.col("Date").dt.month(),
            year = pl.col("Date").dt.year(),
        )
        .with_columns(
            pl.when(pl.col("month") <= 3).then(1)
            .when(pl.col("month") <= 6).then(2)
            .when(pl.col("month") <= 9).then(3)
            .otherwise(4)
            .alias("Quarter")

        )
        .group_by(["year", "Quarter"])
        # compute quarterly production
        .agg(
            pl.col("Production").sum().round(2).alias("Quarterly_Production")
        )
        .sort(["year", "Quarter"])
        .select(
            "year",
            "Quarter",
            "Quarterly_Production"
        )
        .with_columns(
            pl.col("Quarter").replace_strict([1, 2, 3,4], ["Q1", "Q2", "Q3", "Q4"]).alias("Quarter"),
            pl.col("year").cast(pl.Utf8).alias("year")
        )
        #cast year to string
       
        
  
)

    year_quarter_df=q.collect() #execute the lazy computation
    
    fig = px.bar(year_quarter_df, x='year', y='Quarterly_Production', color='Quarter', barmode='group')
    fig.update_layout(xaxis_title="Quarter", yaxis_title="Quarterly Production (kWh)")
    st.plotly_chart(fig)

    ```

    Anyways, this is was a good exercise to learn Polars. I'm definitely looking forward to using Polars in future projects. Feel free to checkout the [source code](https://github.com/indapa/SolarProduction/) and you can see the app in action [here](https://indapa-solar.streamlit.app/).
    