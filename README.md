# Paykun Android SDK V2

Paykun SDK is hosted in maven Central so first confirm that you have 'mavenCentral()' in your repositories section.

```gradle
allprojects {
    repositories {
        mavenCentral()
    }
}
```

## Installation

Add the paykun SDK in your dependencies section

```gradle
dependencies {
    implementation 'com.paykun.sdk:paykun-androidx-sdk:1.1.17'
}
```

## Usage

* Performing transaction

```java
// Initiate Transaction
public void createPayment()
{
	// Required data to be provided
	String merchantIdLive="<Marcent Id>";
	String accessTokenLive="<Mobile Access Token>";
	String productName="<Product Name>";
	String orderId= String.valueOf(System.currentTimeMillis()); // You can change this based on your requirement
	String amount="<Amount>";
	String customerName="<Customer Name>";
	String customerPhone="<Customer Mobile No>";
	String customerEmail="<Customer Email Id>";
	
	// You can customize which payment method should be provided
	// Here is the example for payment method customization, This is optional.
	
	// Create Map for payment methods
	HashMap<PaymentTypes, PaymentMethods> payment_methods = new HashMap<>();
	
	// Create payment method object to be added in payment method Map
	PaymentMethods paymentMethod = new PaymentMethods();
	paymentMethod.enable=true; // True if you want to enable this method or else false, default is true
	paymentMethod.priority=1; // Set priority for payment method order at checkout page
	paymentMethod.set_as_prefered=true; // If you want this payment method to show in prefered payment method then set it to true
	
	// Add payment method into our Map
	payment_methods.put(PaymentTypes.UPI, paymentMethod);
	
	// Example for netbanking
	paymentMethod = new PaymentMethods();
	paymentMethod.enable=true;
	paymentMethod.priority=2;
	paymentMethod.set_as_prefered=true;
	paymentMethod.sub_methods.add(new Sub_Methods(SubPaymentTypes.SBIN,1));
	paymentMethod.sub_methods.add(new Sub_Methods(SubPaymentTypes.HDFC,1));
	payment_methods.put(PaymentTypes.NB, paymentMethod);
	
	// Example for wallet
	paymentMethod = new PaymentMethods();
	paymentMethod.enable=false;
	paymentMethod.priority=3;
	paymentMethod.set_as_prefered=false;
	payment_methods.put(PaymentTypes.WA, paymentMethod);
	
	// Example for card payment
	paymentMethod = new PaymentMethods();
	paymentMethod.enable=true;
	paymentMethod.priority=3;
	paymentMethod.set_as_prefered=true;
	payment_methods.put(PaymentTypes.DCCC, paymentMethod);
	
	// Example for UPI Qr
	paymentMethod = new PaymentMethods();
	paymentMethod.enable=true;
	paymentMethod.priority=3;
	paymentMethod.set_as_prefered=true;
	payment_methods.put(PaymentTypes.UPIQRCODE, paymentMethod);
	
	// Example for EMI
	paymentMethod = new PaymentMethods();
	paymentMethod.enable=true;
	paymentMethod.priority=3;
	paymentMethod.set_as_prefered=true;
	payment_methods.put(PaymentTypes.EMI, paymentMethod);
	
	// Now, Create object for paykun transaction
	PaykunTransaction paykunTransaction=new PaykunTransaction(merchantIdLive,accessTokenLive,true);
	
	try {
		// Set all request data
		paykunTransaction.setCurrency("INR");
		paykunTransaction.setCustomer_name(customerName);
		paykunTransaction.setCustomer_email(customerEmail);
		paykunTransaction.setCustomer_phone(customerPhone);
		paykunTransaction.setProduct_name(productName);
		paykunTransaction.setOrder_no(orderId);
		paykunTransaction.setAmount(amount);
		paykunTransaction.setLive(true); // Currently only live transactions is supported so keep this as true
		
		// Optionally you can customize color and merchant logo for checkout page
		paykunTransaction.setTheme_color("<COLOR CODE>");
		paykunTransaction.setTheme_logo("<LOGO URL>");
		
		// Set the payment methods Map object that we have prepared above, this is optional
		paykunTransaction.setPayment_methods(payment_methods);
		
		new PaykunApiCall.Builder(PaymentActivity.this).sendJsonObject(paykunTransaction);
	} catch (Exception e) {
		e.printStackTrace();
	}
}
```
* Handling transaction callback (Response)

```java
// Handling Payment callback, This Activity/Fragment Must implement PaykunResponseListener interface
// This interface provides callback methods 'onPaymentSuccess' and 'onPaymentError'
public class PaymentActivity extends AppCompatActivity implements PaykunResponseListener 
{
	@Override
    public void onPaymentSuccess(PaymentMessage message) {
        Log.e("onPaymentSuccess",message.toString());
        Toast.makeText(this,"Your Transaction is Success With Id "+message.getTransactionId(),Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onPaymentError(PaymentMessage message) {
        if(message.getResults().equalsIgnoreCase(PaykunHelper.MESSAGE_CANCELLED)){
            // do your stuff here
            Toast.makeText(this,"Your Transaction is cancelled",Toast.LENGTH_SHORT).show();
        }
        else if(message.getResults().equalsIgnoreCase(PaykunHelper.MESSAGE_FAILED)){
            // do your stuff here
            Toast.makeText(this,"Your Transaction is failed",Toast.LENGTH_SHORT).show();
        }
        else if(message.getResults().equalsIgnoreCase(PaykunHelper.MESSAGE_NETWORK_NOT_AVAILABLE)){
            // do your stuff here
            Toast.makeText(this,"Internet Issue",Toast.LENGTH_SHORT).show();

        }else if(message.getResults().equalsIgnoreCase(PaykunHelper.MESSAGE_SERVER_ISSUE)){
            // do your stuff here
            Toast.makeText(this,"Server issue",Toast.LENGTH_SHORT).show();
        }else if(message.getResults().equalsIgnoreCase(PaykunHelper.MESSAGE_ACCESS_TOKEN_MISSING)){
            // do your stuff here
            Toast.makeText(this,"Access Token missing",Toast.LENGTH_SHORT).show();
        }
        else if(message.getResults().equalsIgnoreCase(PaykunHelper.MESSAGE_MERCHANT_ID_MISSING)){
            // do your stuff here
            Toast.makeText(this,"Merchant Id is missing",Toast.LENGTH_SHORT).show();
        }
        else if(message.getResults().equalsIgnoreCase(PaykunHelper.MESSAGE_INVALID_REQUEST)){
            Toast.makeText(this,"Invalid Request",Toast.LENGTH_SHORT).show();
        }
    }
}
```
