# Ontario Agency Data Inventories

Ontario.ca allows you to [search](https://www.ontario.ca/search/data-catalogue) the [data inventories](https://www.ontario.ca/data/government-wide-data-inventory) of ministries only. To get the data inventories of agencies, you need to visit [each agency website](https://www.ontario.ca/page/agency-accountability) and search for its data inventory.

Or, you can [download that information here](/inventories.csv).

**Last updated: March 4, 2018**

# Development

    bundle
    bundle exec rake --tasks
    bundle exec rake agencies
    bundle exec rake inventories > inventories.csv

Copyright (c) 2016 James McKinney, released under the MIT license
