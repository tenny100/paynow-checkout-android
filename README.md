# Paynow Android Library
Paynow Android Library

How to implement this library:

* Copy the lib file to the projects 'libs' folder

* Add the follwing in project build.gradle file

      allprojects {
        repositories {
            .....
            flatDir {
                dirs 'libs'
            }
          }
        }
  
 * In the app build.gradle file, add:
 
        dependencies {
          ...
          compile(name: 'paynow-checkout', ext: 'aar')
        }
  
  * In your Activity add the following pieces of code:
    
        import com.tennyapps.payments.checkout.Checkout;
        import com.tennyapps.payments.checkout.Constants;
        import com.tennyapps.payments.checkout.interfaces.CheckoutInterface;
        import com.tennyapps.payments.checkout.listeners.CheckoutListener;


        public class MainActivity extends Activity implements CheckoutListener {

        private static final int REQUESTCODE = 255;
        private String pollUrl;
        private String id, key;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
           
        ....
        CheckoutInterface.getInstance().setCheckoutListener(this);

        (findViewById(R.id.PaymentButton)).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //declare variables
                Constants.INTEGRATION_ID = "xxxx";
                Constants.INTEGRATION_KEY = "xxxx-xxxx-xxx-xxxxxxxxxx;
                Constants.RESULT_URL = "https://yourserver.com/result/url";
                Constants.RETURN_URL = "https://yourserver.com/return/url";
                Constants.PING_URL = "https://yourserver.com"; // for checking networkconnectivity

                checkout = new Checkout(MainActivity.this);
                checkout.init(Constants.INTEGRATION_ID, Constants.INTEGRATION_KEY);
                checkout.setResultUrl(Constants.RESULT_URL);
                checkout.setReturnUrl(Constants.RETURN_URL);
                checkout.setDescription("Unlock level 2 of some game");
                checkout.setReference("Checkout101");
                checkout.setTitle("Level 2");
                checkout.setAmount(1.99);
                checkout.setAuthEmailAddress("user's email address");
                checkout.createPayment();

                /* Begin Paynow payment process */
                checkout.start(REQUESTCODE);
            }
        });
        
        
        .....
        }


        /* Polls Paynow for payments status after payment */
        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            if (requestCode == REQUESTCODE) {
                if (resultCode == RESULT_OK) {
                    //Check payment status
                    checkout.poll(pollUrl);
                }
            }
        }

        /*
        Payment request error message
         */
        @Override
        public void onInitRequestError(String error) {
            Log.e("initerror", error);
        }

        @Override
        public void onPollRequestError(String error) {
            Log.e("pollerror", error);
        }

        /*
        Network connection error
             */
        @Override
        public void onNetworkError(String error) {
            Toast.makeText(context, error, Toast.LENGTH_LONG).show();
        }

        /*
        Status paid indicated all went well, give the user whatever they purchased
        */
        @Override
        public void onPollRequestSuccess(String reference, String paynowReference, String amount, String status, String pollUrl, String hash) {

        if (status.toLowerCase(Locale.ENGLISH).trim().contentEquals("paid")) {
            // Activate game level 2
        } else {
            Toast.makeText(context, status, Toast.LENGTH_LONG).show();
        }
        }

        /*
        If payment initialization is successful, get the pollUrl
         */
        @Override
        public void onInitRequestSuccess(String status, String redirectUrl, String pollUrl, String hash) {
            this.pollUrl = pollUrl;

        }

        }

