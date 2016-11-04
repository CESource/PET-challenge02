# TU Vienna - 188.982 - Privacy Enhancing Technologies - Challenge 02

## Acknowledgment
This challenge is based on the [PET-Exercises repository of gdanezis](https://github.com/gdanezis/PET-Exercises).


## Warning
Be aware that there is a strict policy concerning plagiarism. There will be (manual and automated) tests against your solution.


## Basic repository operations and testing

To clone this repository, go to the directory where you want it to be located and type:

    $ git clone https://github.com/PETS-TUWien/challenge02.git

To update your local version in the repository, go to the GIT repository directory and type:

    $ git pull


### Structure of the challenge
There are two python files in this repository for this challenge.

- The first is named `Lab02Code.py` and contains the structure of the code you need to complete. 
- The second is named `Lab02Tests.py` and contains unit tests (written for the pytest library) that you may execute to partially check your answers. 

Note that the tests passing is a necessary but not sufficient condition to fulfill each task. There are programs that would make the tests pass that would still be invalid (or blatantly insecure) implementations.

The only dependency your Python code should have, besides pytest and the standard library, is the petlib library.


### Installing the dependencies
To install `petlib` type:

    $ sudo pip install petlib

The petlib documentation is [available on-line here](http://petlib.readthedocs.org/en/latest/index.html).
- Red Hat is currently not supplying Elliptic Curve Crytography in the OpenSSL binary packages in their repositories due to concerns about patents. This causes a SegFault when trying to use the ECDSA functions ([Bugreport on Redhat Bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=319901)).
- The installation can also be tricky on Mac or Windows. 
- In that case the easiest way is to download a fully installed virtual machine of Ubuntu on [OSBoxes.org](https://www.osboxes.org/). Within those VMs you still have to install python, petlib and its dependencies, but it shouldn't cause any problems.
- Make sure that all the pre-requisites for petlib (as described in the documentation) are installed.
- Install py.test using pip or easy_install instead of the PPA of your distribution. This prevents version mismatch errors.


### Working with unit tests
Unit tests are run from the command line by executing the command:

```
$ py.test -v Lab02Tests.py
```

Note the `-v` flag toggles a more verbose output. If you wish to inspect the output of the full tests run you may pipe this command to the `less` utility (execute `$ man less` for a full manual of less):

```
$ py.test -v Lab02Tests.py | less
```

You can also run a selection of tests associated with each task, by executing the command. the argument to the `-m` flag may be a task (from task1 to task4 for challenge 2).

```
$ py.test -v Lab02Tests.py -m task1
```

You may also select tests to run based on their name using the `-k` flag. Have a look at the test file to find out the function names of each test. For example the following command executes the very first test, since it matches its name `test_petlib_present`:

```
$ py.test -v Lab02Tests.py -k petlib
```

The full documentation of pytest is [available here](http://pytest.org/latest/).

## Challenge 02 -- Basics of Engineering Mix Systems and Traffic Analysis

## TASK 1 -- Check installation - 0 Points

> Ensures that the key libraries may be loaded, and the code files are present. Nothing to do beyond ensuring this is the case.
> You don't earn any points for that task because it's just the setup of the environment and also it cannot be tested by us.

## TASK 2 -- Build a simple 1-hop mix client - 4 Points

> You are provided the code of the inner decoding function of a simple, one-hop mix server. Your task is to write a function that encodes a message to be send through the mix.

## Hints:

- You can run the tests just for this task by executing:
```
$ py.test -v Lab02Tests.py -m task2
```

- Your objective is to complete the function `mix_client_one_hop`. This function takes as inputs a public key (an EC element) of the mix, an address and a message. It must then encode the message to be processed by the `mix_server_one_hop` in such a way that the mix will output a tuple of (address, message) to be routed to its final destination.

- The message type is a Python NamedTuple already defined for you as `OneHopMixMessage`. The function `mix_client_one_hop` must return an object of this type. Such an object may be created simply by calling:

	OneHopMixMessage(client_public_key, expected_mac, address_cipher, message_cipher)

	where the `client_public_key` is an EC point, the expected Hmac is an Hmac of the `address_cipher` and `message_cipher`, and those are AES Counter mode (AES-CTR) ciphertexts of the encoded address and message.

- Study the function `mix_server_one_hop` that implements the one-hop mix. Take note of all the `petlib` cryptographic operations and checked performed in order to process a message. You will have to ensure they decode your message correctly.

- The first element of a message is an ephemeral public key defined by the client (and the client knows its private part). The private key is used to derived a shared secret with the mix, using the mix public key. Study the code of the one-hop mix to examine the key derivation, and ensure your client mirrors it to generate messages that decode correctly.

- Study the code of the mix in `mix_server_one_hop` to determine the cryptographic operations necessary to encrypt correctly the address and message, as well as producing a valid Hmac. A helper function `aes_ctr_enc_dec` is provided to help you encrypt/decrypt using AES Counter mode. If you are not familiar with [AES Counter mode have a look online](http://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_.28CTR.29).

- An Hmac is a cryptographic checksum that can be used to ensure parts of a message were generated by the holder of a shared secret key. If you are unsure of what a message authentication code does, do check the [HMAC primitive on wikipedia](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code). You can find functions to generate and check Hmac in `petlib.hmac`.

## TASK 3 -- Build a n-hop mix client - 6 Points (+3 Points Bonus)

> In this exercise you will be required to encode a message to be relayed by a cascade of mixes. Each of the mixes has a public key, and the messages is passed to each of the mixes in order before being output.
>
> Your task here is to complete the `mix_client_n_hop` so that it encodes an address and message to be relayed by a sequence of mixes in order (the sequence is implicit, the order of their public keys is given.)

## Hints:

- You can run the tests just for this task by executing:
```
$ py.test -v Lab02Tests.py -m task3
```

- The key differences between the 1-hop mix and the n-hop mix message encoding relates to: (1) the use of a blinding factor to provide bit-wise unlikability of the public key associated with the message (only when implementing the bonus challenge, see below);  (2) the inclusion of a sequence (list) of hmacs as the second part of the mix message; (3) the decryption of the hmacs (in addition to the address and message) at each step of mixing.

- The output of the function `mix_client_n_hop` should be an `NHopMixMessage`. The first element is a public key, the second is a list of hmacs, the third is the encrypted address and the final one is the encrypted message. You can build such a structure using:

	NHopMixMessage(client_public_key, hmacs, address_cipher, message_cipher)

- Study the mix operation in `mix_server_n_hop`. You will notice that the mix derives a shared key, and then uses it to check the first hmac in the list against the rest of the ciphertext. You must ensure your client encoding passes this test.

- Implementing the blinding factor, which ensures a bit-wise unlikability of the public key, is a bonus challenge. Make sure that the corresponding parameter activates the usage of the blinding factor. If you do not implement it you can ignore the parameter. You can run the tests for the bonus task by executing:
```
$ py.test -v Lab02Tests.py -m task3_bonus
```

- Debugging tip: the blinding factor used to "change" the public key of each message makes this task quite complex. Ensure that the sequence of shared keys the client encoder derives is the same as the sequence of shared keys observed by each mix in order.

- This task is fiddly, and you are likely to see a number of hmac verification failures. Do not lose hope: systematically dump the inputs to your hmacs (using `print`) as well as the keys, to ensure the client and the servers hmac the same data under the same keys.

## TASK 4 -- Simple Traffic Analysis / Statistical Disclosure - 5 Points

> In this exercise you will be required to recover the social contacts of a target user, that is sending messages through an anonymity system (traffic analysis). A trace of traffic is provided to your function, as well as the number of social contacts of the target. You should return your best guess about who the user 0 has sent messages to.

## Hints:

- You can run the tests just for this task by executing:
```
$ py.test -v Lab02Tests.py -m task4
```

- There is no need to use `petlib` for this task. Using other Python facilities such as the `Counter` class (from `collections`) might be helpful to keep your answer short (but not necessary).

- The trace provided is a list of tuples [(Senders, Receivers)]. Each item of the list represents one round of the anonymity system, the senders observed sending in this round, and the receivers observed receiving. The identifiers of the senders / receivers are just small integers.

- Your task is to complete the function `analyze_trace` to return the identifiers of a number of receivers that are the friends (the target has sent messages to) of the target. The number of friends sought is provided as `target_number_of_friends`, and the identifier of the target sender is provided as `target` (by default 0). You must receive a the list of receivers that Alice is sending messages to.

- Do study the function `generate_trace` that simulates a very simple anonymity system. It provides a good guide as to the types of data in the trace and their meaning, as well as a model you analysis can be based on.

- Remember the insight from the Statistical Disclosure Attack: anonymity systems provide imperfect privacy. This is mainly due to the fact that when a target is sending its small number of contacts are more likely to be receiving than other users. You will need to turn this insight into an algorithm that finds those contacts.
