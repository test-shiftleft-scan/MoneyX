## Insecure Direct Object References

#### Description

Insecure Direct Object References are a class of vulnerability where users are able to directly access database objects based on their input. This commonly manifests itself in application URLs that end in an object identifer, such as ```/profile/<user id>```. In many cases, the developer does not consider that simplying not giving the user a link to another page does not prevent them from finding it on their own.

In order for this problem to be an *insecure* direct object reference, it should allow the attacker to bypass normal authentication methods. For example, a system that stores invoices and allows them to be accessed by ID (```/invoice?id=343243```) would have IDOR vulnerabilities if an attacker could bruteforce ID numbers to view other users' private invoices.

#### Code Snippet

com.nVisium.androidnv.api.controller.PaymentController

```
	@RequestMapping(value = "/list-received/{id}", method = RequestMethod.GET)
	public String listReceivedPayments(
			@PathVariable Long id,
			Model model) {
		List<Payment> payments = paymentService.getReceivedPayments(id);
		// ...
	}

```
com.nVisium.androidnv.api.controller.EventController

```
	@RequestMapping(value = "/list-member/{user}", method = RequestMethod.GET)
	public String listEventMembership(
			@PathVariable("user") Long user,
			Model model) {
		java.util.List<EventMembership> events_m = eventService.getEventsByMembership(user);
		// ...
	}
```

#### Problem

In both the ```EventController``` and ```PaymentController```, there is no verification that the user submitting the request is attempting to view their own information. It is reasonable to assume that payments a user has sent, and bills they have left to pay, are to be kept private. Yet, if you pass a user ID that is not yours in the URL, the application will dutifully reveal that user's information. MoneyX has several other instances of IDOR, all of which follow the same basic insecure pattern.

It's important to note that Spring Security does ensure that users have to be logged in to access either of these controller actions. However, that is not enough to prevent IDOR. Since users can directly access database object information by changing the ```userid``` in the URl, this is still IDOR.

#### Solution

In both controllers, rather than prompting the user for a ```userid``` from the URL, simply use the current user's ID. In Spring Security, this can be obtained with ```SecurityContextHolder.getContext().getAuthentication().getId()```. Thus, the user would only be able to access their own bills and sent payments.