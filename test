import os

from dotenv import load_dotenv, find_dotenv

load_dotenv(find_dotenv())

import time
import serial

ser = serial.Serial(
    port='/dev/ttyS2',
    baudrate=9600,
    parity=serial.PARITY_NONE,
    stopbits=serial.STOPBITS_ONE,
    bytesize=serial.EIGHTBITS,
    timeout=1
)

import stripe

stripe.api_key = os.getenv('STRIPE_SECRET_KEY')

reader_id = os.getenv('TERMINAL_ID')

my_currency = "gbp"
my_payment_method_types = ["card_present"]
my_capture_method = "manual"

while 1:
    received_data = ser.read_until('\n').decode()  # read serial port
    time.sleep(0.001)

    if (received_data != ''):
        received_data = received_data.replace('\n', '')
        print(received_data)  # print received data
        if (received_data == '1'):
            # create and process payment intent
            payment_intent_create_response = stripe.PaymentIntent.create(
                amount=330,
                currency=my_currency,
                payment_method_types=my_payment_method_types,
                capture_method=my_capture_method,
            )
            print("payment intent id: " + payment_intent_create_response.id)

            payment_intent_retrieve_response = stripe.PaymentIntent.retrieve(
                payment_intent_create_response.id)
            print("payment intent status: " + payment_intent_retrieve_response.status)

            reader_process_payment_intent_response = stripe.terminal.Reader.process_payment_intent(
                reader_id,
                payment_intent=payment_intent_create_response.id
            )
            print("reader status: " + reader_process_payment_intent_response.status)

            print("reader payment process status: " + reader_process_payment_intent_response.action.status)

            stripe.terminal.Reader.TestHelpers.present_payment_method(
                reader_id,
            )

            time.sleep(2)

            payment_intent_retrieve_response = stripe.PaymentIntent.retrieve(
                payment_intent_create_response.id)
            print("payment intent status: " + payment_intent_retrieve_response.status)

            reader_status = stripe.terminal.Reader.retrieve(reader_id)
            print("reader payment process status: " + reader_status.action.status)

            if (payment_intent_retrieve_response.status == "requires_payment_method"):
                # wait for the client to process payment
                print("payment is not proceeded yet by client")
                ser.write("1")  # transmit data serially
                time.sleep(0.5)
                received_data = ser.read_until('\n').decode()  # read serial port
                time.sleep(0.001)

                if (received_data != ''):
                    received_data = received_data.replace('\n', '')
                    print(received_data)  # print received data
                    if (received_data == '2'):
                        time_thirty_seconds = time.time()
                        while 1:
                            payment_intent_retrieve_response = stripe.PaymentIntent.retrieve(
                                payment_intent_create_response.id)
                            print("payment intent status: " + payment_intent_retrieve_response.status)
                            time.sleep(0.5)
                            if (time.time() - time_thirty_seconds >= 30) or (payment_intent_capture_response.status == "succeeded"):
                                print("breaking loop. reason: 30 seconds or payment done")
                                break

            elif (payment_intent_retrieve_response.status == "requires_capture"):
                payment_intent_capture_response = stripe.PaymentIntent.capture(
                    payment_intent_create_response.id)
                print("payment intent status: " +
                      payment_intent_capture_response.status)
                if (payment_intent_capture_response.status == "succeeded"):
                    print("payment done successfully")
                    ser.write(str.encode('2'))  # transmit data serially
                    #break
                #break
            #break
        #break
    # enjoy in while loop
