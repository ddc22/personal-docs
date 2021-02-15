# Where does the data for the plans page come from in wp-calypso?

## Generic Plans Data?


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

Some generic plans details are taken from the following line
```javascript
const planObject = getPlan( state, planProductId );
```

If we dig deeper into the selector we can see the relevant state key which contains this data. [link](https://github.com/Automattic/wp-calypso/blob/5620b340cf7340755e663bd5125c260204e057d1/client/state/plans/selectors/plan.js#L13-L21)


```javascript
/**
 * Return WordPress plans getting from state object
 *
 * @param {object} state - current state object
 * @returns {Array} WordPress plans
 */
export const getPlans = ( state ) => {
	return state.plans.items;
};
```
If we now look at the adjacent selector we can see the API call which requests this information


</br>
</br>

# How the price details are extracted

The price details can be taken from two places depending on whether we are calling from within a site or not.


With Site Id : [link](https://github.com/Automattic/wp-calypso/blob/d0c7384ea6771e36cf351884dacafc6b8f5eddb1/client/lib/wpcom-undocumented/lib/undocumented.js#L876-L899)

```javascript
/**
 * Get a site specific details for WordPress.com plans
 *
 * @param {Function} siteDomain The site slug
 * @param {Function} fn The callback function
 */
Undocumented.prototype.getSitePlans = function ( siteDomain, fn ) {
	debug( '/sites/:site_domain:/plans query' );

	// the site domain could be for a jetpack site installed in
	// a subdirectory.  encode any forward slash present before making
	// the request
	siteDomain = encodeURIComponent( siteDomain );

	return this._sendRequest(
		{
			path: '/sites/' + siteDomain + '/plans',
			method: 'get',
			apiVersion: '1.3',
		},
		fn
	);
};
```
---

Without site id : it leads to the generic plan call discussed above

```javascript
/**
 * Returns the full plan price if a discount is available and the raw price if a discount is not available
 *
 * @param  {object}  state     global state
 * @param  {number}  productId the plan productId
 * @param  {boolean} isMonthly if true, returns monthly price
 * @returns {number}  plan price
 */
export function getPlanRawPrice( state, productId, isMonthly = false ) {
	const plan = getPlan( state, productId );
	if ( get( plan, 'raw_price', -1 ) < 0 ) {
		return null;
	}
	const price = get( plan, 'orig_cost', 0 ) || plan.raw_price;

	return isMonthly ? calculateMonthlyPriceForPlan( plan.product_slug, price ) : price;
}
```