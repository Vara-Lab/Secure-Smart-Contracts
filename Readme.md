[![Open in Gitpod](https://img.shields.io/badge/Open_in-Gitpod-white?logo=gitpod)](https://gitpod.io/new/#https://github.com/)

# Tutorial: Secure Rust Implementations in Smart Contracts on Vara Network

## Introduction
This tutorial provides a step-by-step guide on how to implement secure, robust, and efficient smart contracts on the Vara Network using Rust. By the end of this tutorial, you will be familiar with the basics of creating and handling custom events and errors, and managing state transitions and validations in your smart contracts.

## Step 1: Create a Public Enum for Events and Errors

Define custom events and errors that your smart contract will use:

```rust
#[derive(Encode, Decode, TypeInfo, Debug)]
#[codec(crate = "gstd::codec")]
#[scale_info(crate = "gstd::scale_info")]
pub enum Event {
    FirstCustomEvent {
        first_field: Id,
        second_field: ActorId,
        third_field: u128,
    },
    SecondCustomEvent {
        first_field: Id,
        second_field: ActorId,
    },
}

#[derive(Debug, Clone, Encode, Decode, TypeInfo)]
#[codec(crate = "gstd::codec")]
#[scale_info(crate = "gstd::scale_info")]
pub enum Error {
    Unauthorized,
    UnexpectedFTEvent,
    MessageSendError,
    NotFound,
    IdNotFoundInAddress,
    IdNotFound,
    NotAdmin,
    NotEnoughBalance
}
 ```

## Step 2: Define the `Handle` Function

Implement the `Handle` function using `InOut<Action, Result<Event, Error>>` to manage inputs and outputs of actions in your smart contract. The `InOut` structure facilitates the management of interactions, allowing a clear handling of the flow of actions and their possible outcomes.

The code below illustrates how to set up the program metadata, specifying the types used for initialization, handling actions, and other relevant aspects of the contract:

```rust
pub struct ProgramMetadata;

impl Metadata for ProgramMetadata {
    // Type used for contract initialization
    type Init = In<Init>;

    // Define 'Handle' to manage actions and results
    type Handle = InOut<Action, Result<Event, Error>>; // Handle the output with `Result<Event, Error>`

    // Other types required by the contract, defined as empty if not used
    type Others = ();
    type Reply = ();
    type Signal = ();
    type State = ();
}
```

## Step 3: Implement the `Handle` Function Architecture to Manage Outputs

The `Handle` function plays a crucial role in managing the behavior of your smart contract by processing incoming actions and generating appropriate outputs. Below is a practical example of how to implement the `Handle` function to handle outputs efficiently:

```rust
let action: Action = msg::load().expect("Could not load Action");

let state = unsafe {
    STATE.as_mut().expect("Unexpected error in state")
};

let result = match action {
    Action::FirstAction { input } => {
        state.first_method(input).await
    },
    Action::SecondAction { input } => {
        state.second_method(input)
    }
};

msg::reply(result, 0).expect("Failed to encode or reply with `Result<Event, Error>`");

   ```

## Step 4: Create Specific Implementations for Actions Using `Result<Event, Error>` Output

In this step, you will define specific functions within your smart contract to handle actions. These functions will perform validations, update the state, and potentially generate events. Below is an example of how to implement a function that handles a specific action and returns a `Result<Event, Error>`:

```rust

impl State {


    fn first_method(&mut self, amount: u128) -> Result<Event, Error> { } // the output with `Result<Event, Error>`


    // More methods...
}
   ```

## Step 5: Add Input Validations in the Implementations

Implementing robust input validations is crucial for maintaining the integrity and security of your smart contract. These validations ensure that only valid and authorized actions are processed. Below is an example of adding input validations within a specific function implementation that returns `Result<Event, Error>`:

```rust
    fn first_method(&mut self, amount: u128) -> Result<Event, Error> {
         // Validations
        let source = msg::source();
        if self.balances.get(&source).unwrap_or(&0) < &amount {
            return Err(Error::NotEnoughBalance);
        }

    // Additional logic and actions follow...
}

```

## Step 6: Follow a Consistent Architecture for Validations, State Transitions, and Events

In this step, you will learn how to implement a consistent framework for handling validations, state transitions, and events in your smart contract. Consistency in these areas ensures that the contract functions reliably and securely. Below is an example of how to structure these elements within a function:

```rust
fn first_method(&mut self, amount: u128) -> Result<Event, Error> {
    // Validations
    let source = msg::source();
    if self.balances.get(&source).unwrap_or(&0) < &amount {
        return Err(Error::NotEnoughBalance);
    }

    // Transition State in Main State
    self.balances
        .entry(source)
        .and_modify(|balance| *balance -= amount);
    self.current_supply -= amount;
    self.total_supply -= amount;

    // Generate event
    Ok(Event::FirstCustomEvent {
        first_field: info_first_field,
        second_field: info_second_field,
        third_field: info_third_field,
    })
}

```

## Step 7: Implement Optional Admin Validations for Smart Contract Functions

Admin validations are a crucial security feature for functions in smart contracts that should be restricted to authorized users only. This step involves adding checks to ensure that only admins can execute certain actions. Below is an example of how to integrate admin validations into a function:

```rust
fn first_method(&mut self, amount: u128) -> Result<Event, Error> {
    // Admin Validations
    let source = msg::source();
    if !self.admins.contains(&source) {
        return Err(Error::NotAdmin);
    }

    // Additional implementation logic follows...
}

```

## Step 8: Add Partial State and Query Functionality in Smart Contract

This step involves implementing a system to query and reply to the state in your smart contract. This is essential for interactions that require state inspection without modifying it.

### 1. Define Enums for Query and Query Reply

Start by defining enums for the types of queries your contract can handle and their corresponding replies:

```rust
#[derive(Encode, Decode, TypeInfo)]
#[codec(crate = "gstd::codec")]
#[scale_info(crate = "gstd::scale_info")]
pub enum Query {
    ExampleQuery,
}

#[derive(Encode, Decode, TypeInfo)]
#[codec(crate = "gstd::codec")]
#[scale_info(crate = "gstd::scale_info")]
pub enum QueryReply {
    ExampleQueryReply(String),
}

```
### 2. Add Query Handling to Program Metadata

Incorporate query handling into your smart contract by defining `InOut<Query, QueryReply>` for the `State` in your `ProgramMetadata`. This allows your contract to process queries and return the corresponding results efficiently.

### Implementation

Modify the `ProgramMetadata` structure to include the `State` type, which will handle both incoming queries and their replies:

```rust
pub struct ProgramMetadata;

impl Metadata for ProgramMetadata {
    type Init = In<Init>;
    type Handle = InOut<Action, Result<Event, Error>>; // Handle actions and their outcomes with `Result<Event, Error>`
    type Others = ();
    type Reply = ();
    type Signal = ();
    type State = InOut<Query, QueryReply>; // Manage queries and their replies
}
```


### 3. Implement Query Handling in the Smart Contract

Integrate query handling functionality to manage and respond to state inquiries within your smart contract. This involves setting up a function that can receive queries, process them, and send back appropriate replies.

### Implementation

The function `state()` is designated to handle queries. Here’s how it is implemented:

```rust
#[no_mangle]
extern "C" fn state() {
    let state = unsafe {
        STATE
            .as_ref()
            .expect("Unexpected: Error in getting contract state")
    };
    let query: Query = msg::load().expect("Unable to decode the query");
    let reply = match query {
        Query::ExampleQuery => QueryReply::ExampleQueryReply(state.field.clone()),
    };
    msg::reply(reply, 0).expect("Error on sharing state");
}
```

## Conclusion

This tutorial has guided you through the essential steps for building secure smart contracts with Rust on the Vara Network. We've explored everything from initializing enums to adding complex validations and admin controls. These foundations not only enhance your smart contract's functionality but also fortify its security.

As you continue to develop in the blockchain space, remember to stay updated with the latest practices and always test thoroughly. Keep building, learning, and contributing to a safer and more robust blockchain ecosystem.
