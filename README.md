# TTG-Tech-Task
I am going to use the Android native programming language **Kotlin** to simulate and implement this task. And we use **Firebase** API- firestore database to save the data.
## Authentication and Interface Overview
Before diving into the specifics of our application, it's crucial to address the initial authentication process. We utilize **Firebase** for user authentication, which seamlessly integrates with our Firestore database. The unique `UserID` plays a pivotal role in this process, although its creation isn't immediately necessary.

## Detail Activity Interface Implementation
After successful login, after the user selects a specific show as merrily we roll along, users are presented with the interface, as depicted in the image below:

![Detail Activity Interface](https://github.com/CrystarRay/TTG-Tech-Task/assets/126846127/d7c220ca-1fe2-4b92-9a16-7da78dc57ef9)


This interface, referred to as the **Detail Activity**, features a key element: the **Enter Lottery** button. Interacting with this button triggers a significant UI change, described in the following section.

### Code Snippet: Enter Lottery Button Functionality

```kotlin
class ShowDetailActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_show_detail)

        // Setup the listener for "Enter Lottery" button
        // This action replaces the current view with the TicketSelectionFragment
        enterLotteryButton.setOnClickListener {
            supportFragmentManager.beginTransaction()
                .replace(R.id.fragment_container, TicketSelectionFragment.newInstance())
                .addToBackStack(null)
                .commit()
        }
    }
}
```
In the `ShowDetailActivity`, we have defined the behavior that occurs when the "Enter Lottery" button is clicked. By utilizing the Android Jetpack's Fragment library, clicking this button initiates a transaction that replaces the current view in the fragment_container with an instance of TicketSelectionFragment. The transaction is also added to the back stack, enabling the user to return to the previous state if they press the back button. 

## Ticket Selection Fragment Implementation

Following the `Detail Activity`, the user navigates to the `TicketSelectionFragment`. This part of the application is pivotal for users to select the number of tickets they wish to purchase.

#### User Interface for Ticket Selection

The user interface for ticket selection is designed for straightforward interactions, as shown in the image below:

![Ticket Selection Interface](https://github.com/CrystarRay/TTG-Tech-Task/assets/126846127/c56f6bc8-7807-4029-8e0c-d8cca303100a)

#### Implementation Details

```kotlin
class TicketSelectionFragment : Fragment() {
    // Companion object remains unchanged

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        val view = inflater.inflate(R.layout.fragment_ticket_selection, container, false)

        // Buttons for selecting the number of tickets
        val oneTicketButton = view.findViewById<Button>(R.id.oneTicketButton)
        val twoTicketsButton = view.findViewById<Button>(R.id.twoTicketsButton)

        oneTicketButton.setOnClickListener {
            navigateToShowtimeSelection(1)
        }

        twoTicketsButton.setOnClickListener {
            navigateToShowtimeSelection(2)
        }

        return view
    }

    private fun navigateToShowtimeSelection(ticketCount: Int) {
        val showtimeSelectionFragment = ShowtimeSelectionFragment.newInstance(ticketCount)

        parentFragmentManager.beginTransaction()
            .replace(R.id.fragment_container, showtimeSelectionFragment)
            .addToBackStack(null)
            .commit()
    }
}

```
In the `ShowtimeSelectionFragment`, we have defined the behavior for dynamically creating showtime pickers based on the number of tickets selected. By utilizing the Fragment's dynamic nature, this fragment adapts to display the appropriate number of showtime pickers. Upon selecting showtimes and clicking the 'Next' button, a transaction is initiated that replaces the current view in the fragment_container with an instance of ContactInformationFragment. This transaction is added to the back stack, allowing the user to return to the previous state by pressing the back button, thus providing a seamless and intuitive navigation experience within the app.
Furthermore, the `ShowtimeSelectionFragment` is designed to be responsive to user choices. It intelligently passes the number of tickets selected — either one or two — to the next fragment in the journey, `ContactInformationFragment`. 

## Showtime Selection Fragment Implementation

Following ticket selection, the ShowtimeSelectionFragment allows users to pick a showtime.

![image](https://github.com/CrystarRay/TTG-Tech-Task/assets/126846127/6e30e19f-8fdb-437f-9fcd-9e57d18ea11e)

#### Implementation Details

```kotlin
class ShowtimeSelectionFragment : Fragment() {
    // Existing factory method remains unchanged

    private var ticketCount: Int = 1

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        arguments?.let {
            //at least 1
            ticketCount = it.getInt("TICKET_COUNT", 1)
        }
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        val view = inflater.inflate(R.layout.fragment_showtime_selection, container, false)

        // Create showtime pickers based on the ticketCount
        for (i in 1..ticketCount) {
            val showtimePicker = createShowtimePicker(I)
            // Add the showtimePicker to the view
            view.addView(showtimePicker)
        }

        view.findViewById<Button>(R.id.nextButton).setOnClickListener {
            // Collect all selected showtimes and pass them to the next fragment
            val selectedShowtimes = (1..ticketCount).map { getSelectedShowtime(it) }
            val contactInformationFragment = ContactInformationFragment.newInstance(selectedShowtimes)

            parentFragmentManager.beginTransaction()
                .replace(R.id.fragment_container, contactInformationFragment)
                .addToBackStack(null)
                .commit()
        }

        return view
    }


```
For `ShowtimeSelectionFragment`, When the user selects showtimes and presses the 'Next' button, the fragment initiates a transition to the ContactInformationFragment. As it uses Android Jetpack's Fragment library to replace the current view in the fragment_container with the new fragment. The transaction is also added to the back stack.In the `ShowtimeSelectionFragment`, the user's choices are key:

1. **Selecting Showtimes**: The user picks showtimes based on how many tickets they chose. For example, if two tickets were selected, they'll pick two showtimes.

2. **Calculating Total Tickets**: The app calculates the total tickets. If the user picks two different times for two tickets, that's 4 tickets in total (2 for each time).

3. **Passing Data to Next Fragment**: This data - the number of tickets and their showtimes - is then sent to the `ContactInformationFragment`. This step is vital for saving the user's choices in the database.

The user must select dates to move to the next fragment, ensuring a logical flow in the app's ticket booking process.


## Contact Information Fragment Implementation
I am assuming we have a final step to let the user enter the contect information and then we can save there data into database to confirm there lottery enter.


#### Implementation Details

```kotlin
class ContactInformationFragment : Fragment() {

    private var ticketCount: Int = 1
    private var showtimes: ArrayList<String> = arrayListOf()

    // Factory method to create a new instance of this fragment with ticket count and showtimes
    companion object {
        fun newInstance(ticketCount: Int, showtimes: ArrayList<String>) = ContactInformationFragment().apply {
            arguments = Bundle().apply {
                putInt("TICKET_COUNT", ticketCount)
                putStringArrayList("SHOWTIMES", showtimes)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        arguments?.let {
            ticketCount = it.getInt("TICKET_COUNT")
            showtimes = it.getStringArrayList("SHOWTIMES") ?: arrayListOf()
        }
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        // Inflate the layout for this fragment
        val view = inflater.inflate(R.layout.fragment_contact_information, container, false)

        // Handle the form submission
        view.findViewById<Button>(R.id.enterButton).setOnClickListener {
            // Collect contact information and save to Firestore
            val fullName = view.findViewById<EditText>(R.id.fullNameEditText).text.toString()
            val phoneNumber = view.findViewById<EditText>(R.id.phoneNumberEditText).text.toString()
            val email = view.findViewById<EditText>(R.id.emailEditText).text.toString()

            // Validate input
            if (validateInput(fullName, phoneNumber, email)) {
                // Prepare data to save
                val userDetails = hashMapOf(
                    "full_name" to fullName,
                    "phone_number" to phoneNumber,
                    "email" to email,
                    "ticket_count" to ticketCount,
                    "showtimes" to showtimes
                )

                // Save to Firestore
                saveToFirestore(userDetails)
            }
        }

        return view
    }

    private fun validateInput(fullName: String, phoneNumber: String, email: String): Boolean {
        // Add validation logic for fullName, phoneNumber, and email
        return fullName.isNotEmpty() && phoneNumber.isNotEmpty() && email.isNotEmpty()
    }

    private fun saveToFirestore(userDetails: HashMap<String, Any>) {
        // Implement Firestore saving logic
        // This is a placeholder for the Firestore API call
    }
}
```
The `ContactInformationFragment` represents the final phase in the lottery entry process. We already have the ticket amount and date passed from the last fragment.

This fragment's primary role is to collect the user's full name, phone number, and email address. Once the user hits the 'Enter' button, it triggers a series of actions:

1. **Validation**: The app first checks to make sure that all the entered information is complete and valid.
2. **Data Preparation**: The user's contact information, along with their ticket count and selected showtimes, are compiled into a `HashMap`. This structure is efficient for organizing the diverse set of data points.
3. **Firestore Integration**: The collected data is then saved to Firestore. This step involves calling the Firestore API, where each user's details are securely stored in a unique data record.
