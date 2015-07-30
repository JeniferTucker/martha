# Manual changes #

Remove left/right margins from grid to allow for vertical alignment of logo and panels down lefthand side of home page and for the panels to align vertically on the rightand side with the edge of the banner.

```
[root@pippa grid]

     pushd /var/local/donkeys-git-local/drupal-7/sites/all/themes/marinelli/css/grid

          vi grid_1000.css

/* Grid >> Global
----------------------------------------------------------------------------------------------------*/

.grid_1,
.grid_2,
.grid_3,
.grid_4,
.grid_5,
.grid_6,
.grid_7,
.grid_8,
.grid_9,
.grid_10,
.grid_11,
.grid_12 {
        display:inline;
        float: left;
        position: relative;
        margin-left: 0px;      /* JLT 20130117 changed from 10px to 0px */
        margin-right: 0px;     /* JLT 20130117 changed from 10px to 0px */
}

/* Grid >> Children (Alpha ~ First, Omega ~ Last)
----------------------------------------------------------------------------------------------------*/

     popd
```

## Avocet theme ##

Applied the above modification to the sub-theme, but this had no effect in the same file.