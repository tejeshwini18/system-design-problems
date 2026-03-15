# LLD: Notify-Me Button Functionality

## 1. Requirements

- **Product** can be out of stock. **User** clicks "Notify Me" (subscribe).
- When product is **back in stock**, all subscribers get **notification** (email/push).
- **Unsubscribe:** User can opt out before notification.
- **One notification per user per product** (don’t spam on every restock event).
- Optional: **priority** (FIFO vs premium first), **channel** (email vs push).

---

## 2. Core Classes (Observer Pattern)

```text
Subject (Product or StockService) – observers: List<NotifyMeObserver>
  attach(observer), detach(observer), notifyObservers()   // when restock

NotifyMeObserver (interface) – update(productId, message)
  EmailNotifyObserver(userId, email), PushNotifyObserver(userId, deviceToken)

NotifyMeService – subscribe(userId, productId, channel?), unsubscribe(userId, productId)
  Store: Map<ProductId, List<Subscription>>   // subscription = userId + channel
  onRestock(productId): getSubscriptions(productId); for each: send notification; mark notified or remove subscription (one-time)
Subscription – userId, productId, channel, createdAt
```

---

## 3. Flow

1. **Subscribe:** User clicks Notify Me on product P. NotifyMeService.addSubscription(userId, productId, channel). Store in DB or cache: productId → list of (userId, channel).
2. **Restock:** Inventory service (or admin) calls NotifyMeService.onRestock(productId). Load subscriptions for productId; for each subscription: send email/push via NotificationService; remove subscription (one-time) or mark as notified so we don’t send again for same restock.
3. **Unsubscribe:** NotifyMeService.removeSubscription(userId, productId).

---

## 4. Design Patterns

- **Observer:** Product (subject) notifies list of observers (subscribers) on restock; observers = notification handlers per user/channel.
- **Strategy:** NotificationStrategy (Email, Push) for different channels.

---

## 5. Key Methods

```text
NotifyMeService.subscribe(userId, productId, channel?)
NotifyMeService.unsubscribe(userId, productId)
NotifyMeService.onRestock(productId)   // internal or called by inventory
```

---

## 6. Data Model

```text
notify_me_subscriptions (user_id, product_id, channel, created_at) UNIQUE(user_id, product_id)
```
- On restock: SELECT * FROM notify_me_subscriptions WHERE product_id = ?; send notifications; DELETE or mark sent.
