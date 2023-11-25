# TTG-Tech-Task
## Authentication and Interface Overview

Before diving into the specifics of our application, it's crucial to address the initial authentication process. We utilize **Firebase** for user authentication, which seamlessly integrates with our Firestore database. The unique `UserID` plays a pivotal role in this process, although its creation isn't immediately necessary.

### Interface Visualization

After successful login, users are presented with the main interface, as depicted in the image below:

![Detail Activity Interface](https://github.com/CrystarRay/TTG-Tech-Task/assets/126846127/d7c220ca-1fe2-4b92-9a16-7da78dc57ef9)

#### Exploring the Detail Activity Interface

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

### Ticket Selection Fragment Implementation

Following the `Detail Activity`, the user navigates to the `TicketSelectionFragment`. This part of the application is pivotal for users to select the number of tickets they wish to purchase.

#### User Interface for Ticket Selection

The user interface for ticket selection is designed for straightforward interactions, as shown in the image below:

![Ticket Selection Interface](https://github.com/CrystarRay/TTG-Tech-Task/assets/126846127/c56f6bc8-7807-4029-8e0c-d8cca303100a)

#### Implementation Details

```kotlin
class TicketSelectionFragment : Fragment() {

    // Factory method to create a new instance of this fragment
    companion object {
        fun newInstance() = TicketSelectionFragment()
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        // Inflate the layout for this fragment
        val view = inflater.inflate(R.layout.fragment_ticket_selection, container, false)

        // Handle ticket selection and navigate to ShowtimeSelectionFragment
        view.findViewById<Button>(R.id.nextButton).setOnClickListener {
            // Pass selected ticket count as an argument to the next fragment
            val selectedTicketCount = view.findViewById<Spinner>(R.id.ticketSpinner).selectedItem.toString().toInt()
            val showtimeSelectionFragment = ShowtimeSelectionFragment.newInstance(selectedTicketCount)

            parentFragmentManager.beginTransaction()
                .replace(R.id.fragment_container, showtimeSelectionFragment)
                .addToBackStack(null)
                .commit()
        }

        return view
    }
}
```

#### Showtime Selection Fragment Implementation

Following ticket selection, the ShowtimeSelectionFragment allows users to pick a showtime.

![Ticket Selection Interface](https://github.com/CrystarRay/TTG-Tech-Task/assets/126846127/c56f6bc8-7807-4029-8e0c-d8cca303100a)

#### Implementation Details

```kotlin
class ShowtimeSelectionFragment : Fragment() {

    companion object {
        fun newInstance(ticketCount: Int) = ShowtimeSelectionFragment().apply {
            arguments = Bundle().apply {
                putInt("TICKET_COUNT", ticketCount)
            }
        }
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        val view = inflater.inflate(R.layout.fragment_showtime_selection, container, false)

        view.findViewById<Button>(R.id.nextButton).setOnClickListener {
            val selectedShowtime = view.findViewById<RadioGroup>(R.id.showtimeRadioGroup).checkedRadioButtonId
            val contactInformationFragment = ContactInformationFragment.newInstance(ticketCount, selectedShowtime)

            parentFragmentManager.beginTransaction()
                .replace(R.id.fragment_container, contactInformationFragment)
                .addToBackStack(null)
                .commit()
        }

        return view
    }
}

```



#### Contact Information Fragment Implementation



#### Implementation Details

```kotlin
class ContactInformationFragment : Fragment() {

    private var ticketCount: Int = 1
    private var showtime: String = ""

    // Factory method to create a new instance of this fragment with ticket count and showtime
    companion object {
        fun newInstance(ticketCount: Int, showtime: String) = ContactInformationFragment().apply {
            arguments = Bundle().apply {
                putInt("TICKET_COUNT", ticketCount)
                putString("SHOWTIME", showtime)
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        arguments?.let {
            ticketCount = it.getInt("TICKET_COUNT")
            showtime = it.getString("SHOWTIME") ?: ""
        }
    }

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        // Inflate the layout for this fragment
        val view = inflater.inflate(R.layout.fragment_contact_information, container, false)

        // Handle the form submission
        view.findViewById<Button>(R.id.enterButton).setOnClickListener {
            // Collect contact information and save to Firestore
            val phoneNumber = view.findViewById<EditText>(R.id.phoneNumberEditText).text.toString()
            val email = view.findViewById<EditText>(R.id.emailEditText).text.toString()

            // Validate input and save data to Firestore
            if (validateInput(phoneNumber, email)) {
                val userDetails = hashMapOf(
                    "phone_number" to phoneNumber,
                    "email" to email,
                    "tickets" to ticketCount,
                    "showtime" to showtime
                )

                // Save to Firestore (assumed to be implemented)
                saveToFirestore(userDetails)
            }
        }

        return view
    }

    private fun validateInput(phoneNumber: String, email: String): Boolean {
        // Implement validation logic
        return true
    }

    private fun saveToFirestore(userDetails: HashMap<String, Any>) {
        // Implement Firestore saving logic
    }
}

```
