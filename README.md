# Demo - Buy Button JS with Custom GraphQL

This demo showcases the buy-button-js with the new **Custom GraphQL** feature enabled.

## 🎯 What's Being Tested

This demo is configured to use **custom GraphQL mutations** with the `requiresShipping` field, which provides:
- **100% accurate** physical vs digital product detection
- **Self-contained** free shipping calculation (no external API calls)
- **Enhanced logging** in browser console

## 🚀 Quick Start

### 1. Open in Browser

Simply open `index.html` in your browser:

```bash
open index.html
```

Or use a local server:

```bash
# Python
python3 -m http.server 8080

# Node.js (if http-server is installed)
npx http-server -p 8080
```

Then visit: http://localhost:8080

### 2. Check Console Logs

Open your browser's Developer Tools (F12) and check the Console. You should see:

```
[Cart] Using custom GraphQL mutations with requiresShipping field
[CustomCartMutations] Initialized with: {domain: "...", apiVersion: "2024-01", hasToken: true}
```

### 3. Test Adding Products

1. Click "Buy From Us" on any product
2. Product will be added to cart
3. Check console for detailed logs:

```
[Cart] Using custom GraphQL mutations
[CustomCartMutations] Executing mutation with variables: {...}
[CustomCartMutations] Successfully added line items
[CustomCartMutations] requiresShipping fields present: {total: 1, withField: 1, withoutField: 0, percentage: 100}
[Cart] Detection method: requiresShipping field (100% accurate)
[Cart]   - requiresShipping field: true
[Cart]   - weight: 1.5 lb
[Cart]   - Final requiresShipping: true
```

## 🧪 Testing Scenarios

### Scenario 1: Physical Products
- Add a book or physical item
- Console should show: `requiresShipping: true`
- Product counts toward free shipping threshold

### Scenario 2: Digital Products
- Add a digital product (if available)
- Console should show: `requiresShipping: false`
- Product excluded from free shipping calculation

### Scenario 3: Mixed Cart
- Add both physical and digital products
- Only physical products count toward shipping threshold
- Check console for detailed calculation logs

## ⚙️ Configuration

The custom GraphQL feature is enabled in `index.html`:

```javascript
cart: {
    useCustomGraphQL: true,  // 🔥 Feature flag enabled
    // ... other options
}
```

### To Disable (Use SDK Mode):

Change to:

```javascript
cart: {
    useCustomGraphQL: false,  // or simply omit this line
    // ... other options
}
```

## 📊 What to Look For

### Success Indicators:

✅ **Console shows:**
- `[Cart] Using custom GraphQL mutations with requiresShipping field`
- `[Cart] Detection method: requiresShipping field (100% accurate)`

✅ **Network tab shows:**
- POST requests to `/api/2024-01/graphql.json`
- Response includes `requiresShipping` field

✅ **Cart behavior:**
- Add to cart works correctly
- Free shipping calculation accurate
- No console errors

### Troubleshooting:

❌ **If you see:** `[Cart] Using standard SDK mutations`
- Feature flag not enabled - check `useCustomGraphQL: true` in cart config

❌ **If you see:** `[Cart] Detection method: weight-based`
- requiresShipping field not available - check API version or token permissions

## 🔄 Rebuilding

If you make changes to buy-button-js source code:

```bash
cd ../buy-button-js

# Start watch mode (auto-rebuild on changes)
yarn run src:watch

# In another terminal, copy to demo
cp tmp/buybutton.dev.js ../demo/buybutton.min.js
```

The watch mode will automatically rebuild when you save files.

## 📝 Files

- `index.html` - Demo page with custom GraphQL enabled
- `buybutton.min.js` - Buy Button JS build (862KB)
- `README.md` - This file

## 🔗 Related Documentation

- [Implementation Summary](../IMPLEMENTATION_SUMMARY.md)
- [Testing Guide](../TESTING_GUIDE.md)
- [Task Checklist](../TASK_CHECKLIST_REVISED.md)
- [Buy Button README](../buy-button-js/readme.md)

## 💡 Tips

1. **Keep Console Open**: All the detailed logging happens in browser console
2. **Clear Cart**: Use `localStorage.clear()` in console to reset cart
3. **Compare Modes**: Toggle `useCustomGraphQL` to compare SDK vs Custom GraphQL behavior
4. **Network Monitoring**: Watch Network tab for GraphQL mutations to Shopify API

## 🎉 Success!

If you see the console logs showing `requiresShipping field (100% accurate)`, the custom GraphQL implementation is working correctly! 🚀
