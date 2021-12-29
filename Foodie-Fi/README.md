# Case Study 3 - Foodie Fi
![Foodie Fi](https://8weeksqlchallenge.com/images/case-study-designs/3.png)
## Entity Relationship Diagram
<img width="389" alt="Foodie Fi - ER Diagram" src="https://user-images.githubusercontent.com/93120413/147627286-254bb9b4-b640-4ab9-ac52-40829d1344d2.png">

##  Data Sets
### Table 1: Plans
- Customers can choose which plans to join Foodie-Fi when they first sign up.
- Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90.
- Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.
- Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
- When customers cancel their Foodie-Fi service - they will have a churn plan record with a null price but their plan will continue until the end of the billing period.

<img width="117" alt="Foodie Fi - Plans" src="https://user-images.githubusercontent.com/93120413/147627288-d16c26b9-8c0c-4848-9893-ce4554afcfb8.png">

### Table 1: Subscriptions
- Customer subscriptions show the exact date where their specific plan_id starts.
- If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the start_date in the subscriptions table will reflect the date that the actual plan changes.
- When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.
- When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.

<img width="92" alt="Foodie Fi - Subscriptions" src="https://user-images.githubusercontent.com/93120413/147627289-79839ae8-0687-4750-8b0e-5a3a5f1872e2.png">
