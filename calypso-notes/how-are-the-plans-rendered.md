## Where does the data for the plans page come from in wp-calypso?

The first time the data store is accessed to get plans information is [here](https://github.com/Automattic/wp-calypso/blob/46128b8ea30dc381fa60e4d27264d7e10888464a/client/my-sites/plan-features/index.jsx#L862-L881)

```javascript
		let planProperties = compact(
			map( plans, ( plan ) => {
				let isPlaceholder = false;
				const planConstantObj = applyTestFiltersToPlansList( plan, abtest );
				const planProductId = planConstantObj.getProductId();
				const planObject = getPlan( state, planProductId );
				const isLoadingSitePlans = selectedSiteId && ! sitePlans.hasLoadedFromServer;
				const showMonthly = ! isMonthly( plan );
				const availableForPurchase = isInSignup
					? true
					: canUpgradeToPlan( state, selectedSiteId, plan ) && canPurchase;
				const relatedMonthlyPlan = showMonthly
					? getPlanBySlug( state, getMonthlyPlanByYearly( plan ) )
					: null;
				const popular = popularPlanSpec && planMatches( plan, popularPlanSpec );
				const newPlan = isNew( plan ) && ! isPaid;
				const bestValue = isBestValue( plan ) && ! isPaid;
				const currentPlan = sitePlan && sitePlan.product_slug;
				const planPath = getPlanPath( plan ) || '';

```
Specifically the following line
```javascript
const planObject = getPlan( state, planProductId );
```

If we dig deeper into the selector we can see the relevant state key which contains this data. [link](https://github.com/Automattic/wp-calypso/blob/5620b340cf7340755e663bd5125c260204e057d1/client/state/plans/selectors/plan.js#L13-L21)
