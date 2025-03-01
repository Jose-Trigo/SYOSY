# SYOSY


 coisas

 Test 1 – QoS=0 & no Persistence (Publish / Subscribe QoS: 0)
Notice that since ‘-c’ flag is omitted, default behaviour is non-persistent connection.
Open two new terminals:
Start Sub:
mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 0
Start Pub:
mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 1" -q 0
The subscriber should receive a message.
Take some time to analyze the received message contents on Wireshark.
Now:
CTRL-C on pub and sub.
Restart Sub:
mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 0
Disconnect.
Start Pub:
mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 1" -q 0
Restart Sub. Did you receive anything? Why?


 in the first test, when the subscriber is connected adn the publisher publishes the broker receives and send the message,

 in the second part of the test, when we disconnect the subscriber, when the publisher publishes, the message is received by the broker, but no subscriber is subscribed so no message is sent, so the subscriber receives nothing.


//////////////////////////



Test 2 – QoS=0 & Persistence (‘-c’ flag indicates persistent connection)
Start Sub:
mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 0 -c
Disconnect.
Start Pub:
mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 2" -q 0
Did it work now? What can be happening?

even tough we have a persistent connection of the subscriber to the broker and the topic is maintened, the QoS of the subscriber is 0 so the broker QoS is dictated by the publisher, so the broker QoS is also 0 so the message is not stored, so when the subscriber connects again, even tough the topic always existed, the message was not stored and the subscriber gets no message.


///////////////////////////

Test 3 – QoS=1 & Persistence
Start Sub: mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 1 -c
Disconnect.
Start Pub: mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 3" -q 1
Reconnect Sub. Analyse the result. What changed and why? Try publishing multiple messages while
the Sub is disconnected.
Retry, but this time without the persistence flag in the Sub. What happens?

Both subscriber and Publisher have QoS 1, meaning that opposed to test 2, the broker stores the message that was published and when the subscriber connects again, the stored messages are sent to the susbscriber.







Deepseek answer 

"Test 1 – QoS=0 & No Persistence (-c flag omitted)

Setup:

    Subscriber: mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 0

    Publisher: mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 1" -q 0

Your Analysis:

    First Run: Subscriber receives the message while connected. ✅

    Second Run: Subscriber disconnects, publisher sends message, subscriber reconnects and receives nothing. ✅

Corrections/Clarifications:

    No Persistence (-c omitted):

        By default, mosquitto_sub uses a non-persistent (clean) session (clean_session=true).

        When the subscriber disconnects, the broker immediately discards the subscription (no retained state).

        Messages with QoS 0 are not stored by the broker, even transiently.

    Why Subscriber Receives Nothing:

        When the publisher sends a QoS 0 message while the subscriber is disconnected, the broker does not retain it (QoS 0 = no delivery guarantees).

        When the subscriber reconnects, the broker no longer has the subscription (non-persistent session) and no stored message.

Test 2 – QoS=0 & Persistence (-c flag used)

Setup:

    Subscriber: mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 0 -c

    Publisher: mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 2" -q 0

Your Analysis:

    Subscriber reconnects but receives nothing because QoS 0 messages are not stored. ✅

Corrections/Clarifications:

    Role of -c Flag:

        The -c flag enables a persistent session (clean_session=false).

        The broker retains the subscriber’s subscription (not messages!) across disconnects.

        However, QoS 0 messages are never stored, even in persistent sessions.

    Why Subscriber Still Receives Nothing:

        Persistent sessions preserve subscriptions but do not store QoS 0 messages.

        The publisher’s QoS 0 message is delivered only to currently connected subscribers.

Test 3 – QoS=1 & Persistence (-c flag used)

Setup:

    Subscriber: mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 1 -c

    Publisher: mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 3" -q 1

Your Analysis:

    Subscriber receives stored messages when reconnecting. ✅

    Without -c, subscriber receives nothing. ✅

Corrections/Clarifications:

    QoS 1 Behavior:

        The broker stores QoS 1 messages for offline subscribers in a persistent session (-c).

        When the subscriber reconnects, the broker delivers all queued messages (up to max_queued_messages in the broker config).

    Without -c (Non-Persistent Session):

        clean_session=true forces the broker to discard the subscription and all queued messages.

        Subscriber receives nothing because the session state is deleted.

    Multiple Messages While Disconnected:

        The broker will queue all QoS 1 messages sent while the subscriber is offline (if using -c).

        Messages are delivered in order when the subscriber reconnects.

Key Takeaways

    QoS Level:

        QoS 0: No storage, no retries, no guarantees.

        QoS 1: Broker stores messages for offline subscribers (if session is persistent).

    Session Persistence (-c flag):

        -c = clean_session=false: Broker retains subscriptions and queued messages (for QoS ≥1).

        No -c = clean_session=true: Broker discards subscriptions and messages on disconnect.

    Interaction of QoS and Persistence:

        Messages are stored only if:

            QoS ≥1 (for delivery guarantees).

            Session is persistent (-c flag).

Your Answers: Final Verdict

    Test 1: Correct. ✅

    Test 2: Correct, but clarify that -c preserves subscriptions, not messages (QoS 0 messages are never stored). ✅

    Test 3: Correct, but emphasize that -c preserves the session state (subscriptions + queued messages). ✅

Well done! Minor terminology tweaks are needed (e.g., "persistent connection" → "persistent session"), but your understanding of MQTT fundamentals is accurate."



TEXT REFORMULATED

"Test 1 – QoS=0 & no Persistence (Publish/Subscribe QoS: 0)

Scenario:
The subscriber starts with mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 0 (no -c flag, defaulting to a non-persistent session). The publisher sends a message with mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 1" -q 0.

First Run:
When the subscriber is connected, the broker receives the QoS 0 message and forwards it immediately. The subscriber receives the message as expected.

Second Run:
If the subscriber disconnects, restarts, and the publisher sends a message while the subscriber is offline, the broker discards the QoS 0 message. This happens because QoS 0 guarantees no message storage, even transiently. When the subscriber reconnects, it receives nothing because (1) the message was never stored and (2) the non-persistent session (clean_session=true) erased the subscription on disconnect.

Conclusion:
QoS 0 + non-persistent sessions mean messages are delivered only to currently connected subscribers. Disconnected subscribers miss messages permanently.

──────────────────────────────────────────────────
Test 2 – QoS=0 & Persistence (-c flag)

Scenario:
The subscriber starts with mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 0 -c (persistent session via clean_session=false). The publisher sends a message with mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 2" -q 0.

Observations:
Even with the -c flag (persistent session), the subscriber receives nothing after reconnecting. While the -c flag preserves the subscription across disconnects, QoS 0 messages are never stored by the broker. When the publisher sends the message to an offline subscriber, the broker discards it immediately.

Conclusion:
Persistent sessions (-c) retain subscriptions but do not store QoS 0 messages. QoS 0 messages are always transient, regardless of session type.

──────────────────────────────────────────────────
Test 3 – QoS=1 & Persistence

Scenario:
The subscriber starts with mosquitto_sub -h localhost -i Sub1 -t "test/message" -q 1 -c (persistent session + QoS 1). The publisher sends a message with mosquitto_pub -h localhost -t "test/message" -m "MESSAGE 3" -q 1.

With -c Flag:
When the subscriber disconnects and the publisher sends a QoS 1 message, the broker stores it (up to max_queued_messages in its config). On reconnecting, the subscriber receives all stored messages. Multiple messages sent while offline are delivered in order.

Without -c Flag:
If the subscriber omits -c (non-persistent session), the broker deletes the subscription and all queued messages on disconnect. Reconnecting results in no messages being delivered.

Conclusion:
QoS 1 + persistent sessions (-c) ensure messages are stored for offline subscribers. Non-persistent sessions discard both subscriptions and messages, even at QoS 1."

