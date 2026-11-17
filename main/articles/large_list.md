# Tips when exporting large listing

## Introduction

For submission, sometimes listings are required. Sometimes large
listings can include thousands of patients and tens of thousands of
rows. Exporting such large listing files is time-consuming and
memory-heavy, and it is not obvious if the job is still ongoing or is
already dead. The time cost is mostly due to paginating the listing
object into multiple pages.

Consider the following example:

\
[`library`](https://rdrr.io/r/base/library.html)`(`[`rlistings`](https://pharmaverse.github.io/rlistings/)`)`\
\
`iris2`` ``<-`` `[`do.call`](https://rdrr.io/r/base/do.call.html)`(``rbind``, `[`rep`](https://rdrr.io/r/base/rep.html)`(`[`list`](https://rdrr.io/r/base/list.html)`(``iris``)``, ``40``)``)`\
`rlist`` ``<-`` `[`as_listing`](https://pharmaverse.github.io/rlistings/reference/listings.md)`(``iris2``,`\
`  key_cols ``=`` ``"Species"``,`\
`  disp_cols ``=`` `[`c`](https://rdrr.io/r/base/c.html)`(``"Sepal.Length"``, ``"Sepal.Width"``, ``"Petal.Width"``, ``"Petal.Length"``)`\
`)`\
\
`bench``::``mark``(`\
`  a ``=`` `[`paginate_to_mpfs`](https://pharmaverse.github.io/formatters/latest-tag/reference/paginate_indices.html)`(``rlist``[``1``:``1000``, ``]``)``,`\
`  b ``=`` `[`paginate_to_mpfs`](https://pharmaverse.github.io/formatters/latest-tag/reference/paginate_indices.html)`(``rlist``[``1``:``2000``, ``]``)``,`\
`  c ``=`` `[`paginate_to_mpfs`](https://pharmaverse.github.io/formatters/latest-tag/reference/paginate_indices.html)`(``rlist``[``1``:``3000``, ``]``)``,`\
`  d ``=`` `[`paginate_to_mpfs`](https://pharmaverse.github.io/formatters/latest-tag/reference/paginate_indices.html)`(``rlist``[``1``:``4000``, ``]``)``,`\
`  e ``=`` `[`paginate_to_mpfs`](https://pharmaverse.github.io/formatters/latest-tag/reference/paginate_indices.html)`(``rlist``[``1``:``5000``, ``]``)``,`\
`  f ``=`` `[`paginate_to_mpfs`](https://pharmaverse.github.io/formatters/latest-tag/reference/paginate_indices.html)`(``rlist``[``1``:``6000``, ``]``)``,`\
`  check ``=`` ``FALSE``,`\
`  max_iterations ``=`` ``1`\
`)`

It gives the following benchmark results:

      expression      min median `itr/sec` mem_alloc `gc/sec` n_itr  n_gc total_time
      <bch:expr> <bch:tm> <bch:>     <dbl> <bch:byt>    <dbl> <int> <dbl>   <bch:tm>
    1 a             6.13s  6.13s    0.163     2.17GB    0.816     1     5      6.13s
    2 b            13.35s 13.35s    0.0749    8.63GB    1.42      1    19     13.35s
    3 c            22.48s 22.48s    0.0445   19.38GB    1.74      1    39     22.48s
    4 d            36.73s 36.73s    0.0272   34.41GB    1.80      1    66     36.73s
    5 e            44.88s 44.88s    0.0223   53.72GB    1.63      1    73     44.88s
    6 f             1.01m  1.01m    0.0166   77.32GB    1.72      1   104      1.01m
    # ℹ 4 more variables: result <list>, memory <list>, time <list>, gc <list>

It is obvious that the time consumption grows linearly as the rows of
data increase.

![](rlisting_time.png)

To decrease the runtime and prevent memory issues, we would recommend
splitting the listing object by a grouping variable and exporting the
output separately. Consider the following demonstration:

\
`iris3`` ``<-`` `[`cbind`](https://rdrr.io/r/base/cbind.html)`(``iris2``, gp ``=`` `[`rep`](https://rdrr.io/r/base/rep.html)`(`[`c`](https://rdrr.io/r/base/c.html)`(``1``, ``2``, ``3``, ``4``, ``5``, ``6``)``, ``1000``)``)`\
`rlist3`` ``<-`` `[`as_listing`](https://pharmaverse.github.io/rlistings/reference/listings.md)`(``iris3``,`\
`  key_cols ``=`` ``"Species"``,`\
`  disp_cols ``=`` `[`c`](https://rdrr.io/r/base/c.html)`(``"Sepal.Length"``, ``"Sepal.Width"``, ``"Petal.Width"``, ``"Petal.Length"``)`\
`)`\
\
`start.time`` ``<-`` `[`Sys.time`](https://rdrr.io/r/base/Sys.time.html)`(``)`\
`rlist3`` ``|>`\
`  `[`split`](https://rdrr.io/r/base/split.html)`(``rlist3``$``gp``)`` ``|>`\
`  `[`lapply`](https://rdrr.io/r/base/lapply.html)`(``FUN ``=`` ``paginate_to_mpfs``)`\
`end.time`` ``<-`` `[`Sys.time`](https://rdrr.io/r/base/Sys.time.html)`(``)`\
\
`time.taken`` ``<-`` ``end.time`` ``-`` ``start.time`\
`time.taken`\
\
`# > Time difference of 36.06119 secs`

In principle, you could consider processing this work with
multi-threading to further reduce the runtime.

\
[`library`](https://rdrr.io/r/base/library.html)`(``parallel``)`\
\
`start.time`` ``<-`` `[`Sys.time`](https://rdrr.io/r/base/Sys.time.html)`(``)`\
`rlist3`` ``|>`\
`  `[`split`](https://rdrr.io/r/base/split.html)`(``rlist3``$``gp``)`` ``|>`\
`  `[`mclapply`](https://rdrr.io/r/parallel/mclapply.html)`(``FUN ``=`` ``paginate_to_mpfs``)`\
`end.time`` ``<-`` `[`Sys.time`](https://rdrr.io/r/base/Sys.time.html)`(``)`\
\
`time.taken`` ``<-`` ``end.time`` ``-`` ``start.time`\
`time.taken`\
\
`#> Time difference of 18.20406 secs`
