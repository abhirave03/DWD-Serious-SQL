# Case Study 5 - Clique Bait
![Clique Bait](https://8weeksqlchallenge.com/images/case-study-designs/7.png)

##  Data Sets
### Table 1: Users
Customers who visit the Clique Bait website are tagged via their cookie_id.

![Clique Bait - Users](https://user-images.githubusercontent.com/93120413/147914457-c5cb920f-9511-43f0-90e8-385994702628.jpg)

### Table 2: Events
Customer visits are logged in this events table at a cookie_id level and the event_type and page_id values can be used to join onto relevant satellite tables to obtain further information about each event. The sequence_number is used to order the events within each visit.

![Clique Bait - Events](https://user-images.githubusercontent.com/93120413/147914473-75dca9ad-dc01-4d13-85ad-a9a1e5f51ccf.jpg)

