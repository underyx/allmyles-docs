=========
 Flights
=========

---------
 Summary
---------

The flight booking/ticket creation workflow consists of five steps.

 1. :ref:`Flight_Search`
 2. :ref:`Flight_Details`
 3. :ref:`Flight_Booking`
 4. :ref:`Flight_Payment`
 5. :ref:`Flight_Ticketing`

Additional calls that are available:

 - :ref:`Flight_Ticketing_Status`
 - :ref:`Flight_Rules`
 - :ref:`Get_Booking`
 - :ref:`Cancel`

.. _Flight_Search:

--------
 Search
--------

Request
=======

.. http:post:: /flights

    Searches for flights that match provided criteria.

    .. note::
        In most cases you'll want to pass 00:00:00 as time for both your
        departure and your return date. Time filtering constraints will be
        very strict otherwise, often resulting in no matches for your query.

    .. note::
        If you plan to present the results of :ref:`Flexible_Date_Search` and
        regular search at the same time to your users, you have two options.

        1. You send both requests in one session - you can only send the second request when you already have the results of the first one.
        2. You send the two requests in separate sessions - in this case you have to include :ref:`Flexible_Date_Search_Reference` in the regular search request, and set `to_be_referenced` to `True` in the flexible date search request.

    :JSON Parameters:
        - **fromLocation** (*String*) -- departure location, given as IATA code
        - **toLocation** (*String*) -- destination, given as IATA code
        - **departureDate** (*String*) -- date of departure, in ISO format,
          including a time code, even though whole day will be searched by
          default
        - **returnDate** (*String*) -- *(optional)* date of return, in ISO
          format, including a time code, even though whole day will be
          searched by default
        - **persons** (:ref:`Person`) -- a list of passengers, grouped by type
          code, containing Persons
        - **userData** (:ref:`User_Data`) -- information about the end user
        - **fromAirport** (*String*) -- *(optional)* departure airport, given
          as IATA code, must be in the city specified in ``fromLocation``
        - **toAirport** (*String*) -- *(optional)* destination airport, given
          as IATA code, must be in the city specified in ``toLocation``
        - **providerType** (*String*) -- *(optional)* type of results to
          retrieve
        - **preferredAirlines** (*String \[ \]*) -- *(optional)* list of
          airlines to filter results to, given as their two character IATA code
        - **extraDays** (*Integer*) -- *(optional)* number of days to call
          :ref:`Flexible_Date_Search` with, between 1-3
        - **options** (:ref:`Options`) -- *(optional)* sorting and filtering options
        - **flexible_date_search_reference** (:ref:`Flexible_Date_Search_Reference`) --
          *(only in case of choosing option 2 described in the note above)* data about
          the flexible date search made with the same parameters as the regular one
        - **to_be_referenced** (*Boolean*) -- *(optional)* `True` if this is a flexible
          date search and a regular search is to be called next with `flexible_date_search_reference`
        - **number_of_bags** (*Integer*) -- *(optional)* The number of bags to be bundled with the price of LCC flights. **This option has no effect for searches with the default provider, please contact Allmyles for details on alternative providers.** 
        - **baggage_charges** (*Boolean*) -- *(optional)* Wheter or not you would like to receive the baggage price tiers in the search step of LCC flights. Baggage price tiers are always sent in the details step, only request this data if you are using it on search. **This option has no effect for searches with the default provider, please contact Allmyles for details on alternative providers.** 
        - **check_in_charges** (*Boolean*) -- *(optional)* Wheter or not you would like to receive the check in price tiers in the search step of LCC flights. **This option has no effect for searches with the default provider, please contact Allmyles for details on alternative providers.** 
        - **speedy_boarding_charges** (*Boolean*) -- *(optional)* Wheter or not you would like to receive the speedy boarding fee information in the search step of LCC flights. **This option has no effect for searches with the default provider, please contact Allmyles for details on alternative providers.** 

.. _Person:

Person
------

    :JSON Parameters:
        - **passengerType** (*String*) -- one of :ref:`PassengerTypes`
        - **quantity** (*Integer*) -- number of travelers of ``passengerType``

.. _PassengerTypes:

PassengerTypes
--------------

    One of ``ADT``, ``CHD`` or ``INF``

.. _Options:

Options
-------

    :JSON Parameters:
        - **sort** (*String*) -- one of :ref:`Sorting Options`
        - **filter** (:ref:`Filters`) -- filtering options

.. _sorting_options:

Sorting Options
---------------

    One of ``total_fare``, ``-total_fare``, ``comfort_score`` or ``-comfort_score``
    (:ref:`Comfort score`). Reverse-order sorting is indicated with a ``-`` sign
    (e.g. ``-total_fare`` would return the most expensive option first).

.. _Filters:

Filters
-------

    :JSON Parameters:
        - **cabin** (*String*) -- one of :ref:`Cabin types`. Filtering for a certain
          cabin returns combinations that contain at least one leg with the desired
          cabin type.

.. _cabin_types:

Cabin types
-----------

    One of ``economy``, ``premium economy``, ``business`` or ``first``

.. _User_Data:

User Data
---------
    :JSON Parameters:
        - **ip** (*String*) -- the end user's IP address, e.g. ``12.123.45.67.``
        - **browser_agent** (*String*) -- the end user's browser agent based on
          the User-Agent HTTP header, e.g.
          ``Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/21.0``

.. _Flexible_Date_Search_Reference:

Flexible Date Search Reference
------------------------------
    :JSON Parameters:
       - **cookie** (*String*) -- the Cookie sent in the header of
         the referenced flexible date search
       - **extra_days** (*Integer*) -- number of days submitted in **extraDays** in
         the referenced flexible date search

Response Body
=============

    :JSON Parameters:
        - **flightResultSet** (:ref:`flight-result` *\[ \]*) -- root container

.. _flight-result:

FlightResult
------------

    .. warning::
        The ``total_fare`` field here does not include the credit card
        surcharge just yet, as fetching the exact surcharge for a specific
        flight can require an extra 5-10 second call to the external provider.

        This surcharge is retrieved in the _`FlightDetails` call.

    .. warning::
        The prices returned in the fields **total_fare** and **ticketing_fee** are
        converted to HUF by default if the provider returns them in a different
        currency. When displaying prices to the user, please refer to
        **price_charged_by_provider** for a more accurate fare, where the total fare
        is returned in the currency the airline is charging, or
        **total_fare_in_preferred_currencies** for prices converted from the
        original currency. **Important**: this price might change later
        as it is not yet updated with credit card and other surcharges.

    :JSON Parameters:
        - **breakdown** (:ref:`Breakdown` *\[ \]*) -- summary of passenger data
          per type
        - **currency** (*String*) -- currency of all prices in response
        - **ticketing_fee** (*Float*) -- fee charged for ticketing
        - **total_fare** (*Float*) -- total fare, including service fee and ticketing fee
        - **combinations** (:ref:`Combination` *\[ \]*) -- list of combination
          objects
        - **total_fare_in_preferred_currencies** (*\[ \]*) -- total fare converted
          to the client's preferred currencies, including service fee and ticketing fee

          - **currency** (*String*)
          - **total_fare** (*Float*)
        - **ticketing_fee_in_preferred_currencies** (*\[ \]*) -- ticketing fee converted
          to the client's preferred currencies, including service fee and ticketing fee

          - **currency** (*String*)
          - **ticketing_fee** (*Float*)
        - **price_charged_by_provider** (*\[ \]*) -- fare and ticketing fee in the currency
          the airline is charging

          - **currency** (*String*)
          - **total_fare** (*Float*)
          - **ticketing_fee** (*Float*)
          
        - **baggageTiers** (:ref:`BaggageTier` *\[ \]*) -- contains the different options the passenger has for bringing baggages along. May be requested to be included for LCC flights, otherwise not included in the results.
        - **speedy_boarding_fee** (:ref:`Price`) -- Only included in LCC results, and only when requested.
        - **check_in_charges** (:ref:`CheckInCharges` *\[ \]*) -- Only included in LCC results, and only when requested.

.. _Breakdown:

Breakdown
---------

    :JSON Parameters:
        - **fare** (*Float[ ]*) -- total price of the tickets for passengers of
          ``type`` (including tax)
        - **tax** (*Float[ ]*) -- total tax on the tickets for passengers of
          ``type``
        - **type** (*String*) -- type of passengers the breakdown is for, see
          (see :ref:`PassengerTypes`)
        - **quantity** (*Integer*) -- number of passengers of ``type``
        - **ticketDesignators** (:ref:`TicketDesignator` *\[ \]*) -- ticket
          designators applicable for passengers of ``type``
        - **fare_in_preferred_currencies** (*\[ \]*) -- fare converted
          to the client's preferred currencies
          - **currency** (*String*)
          - **fare** (*Float*)
          - **tax** (*Float*)

.. _TicketDesignator:

TicketDesignator
----------------

    Ticket designators are the mini-rules for the flight, with entries such as
    ``{"code": "70|PEN", "extension": "TICKETS ARE NON-REFUNDABLE|"}``.

    :JSON Parameters:
        - **code** (*String*) -- ticket designator's code
        - **extension** (*String*) -- ticket designator's description

.. _Combination:

Combination
-----------

    Combinations are the sets of different flight itineraries that can be
    booked. Every combination in a flight result is guaranteed to have the
    same total price, but the departure times, arrival times, and transfer
    locations can differ.

    :JSON Parameters:
        - **bookingId** (*String*) -- the unique identifier of this
          combination (this is later used to identify the combination when
          booking, for example.)
        - **firstLeg** (:ref:`Leg`) -- the outbound leg of the itinerary
        - **returnLeg** (:ref:`Leg`) -- the inbound leg of the itinerary
        - **serviceFeeAmount** (*Float*) -- ticket designator's description
        - **comfortScore** (:ref:`Comfort score`) -- the comfort score of
          the combination
        - **service_fee_in_preferred_currencies** (*\[ \]*) -- service fee
          converted to the client's preferred currencies
          - **currency** (*String*)
          - **service_fee** (*Float*)

.. _Leg:

Leg
---

    Legs are made up of one or more segments, and span from one location the
    customer searched for to the other.

    :JSON Parameters:
        - **elapsedTime** (*String*) -- The total time between the leg's first
          departure, and last arrival (including time spent waiting when
          transferring). It is given in the format ``HHMM``.
        - **flightSegments** (:ref:`Segment` *\[ \]*) -- The list of segments
          this leg is made up of.

.. _Segment:

Segment
-------

    Segments are the smallest unit of an itinerary. They are the direct
    flights the passenger will take from one stop to another.

    :JSON Parameters:
        - **departure** (:ref:`Stop`) -- data about the flight's departure
        - **arrival** (:ref:`Stop`) -- data about the flight's arrival
        - **aircraft** (*String*) -- Planned aircraft scheduled for the
          specific segment
        - **availableSeats** (*Integer*) -- the number of seats available for
          this price tier---the maximum number we can know of is 9, so when 9
          is returned, that means 9 or more seats are available.
        - **cabin** (*String*) -- one of 'economy', 'first', or 'business'
        - **class** (*String*) -- an airline-specific identifier used in fare
          pricing. The code related to comfort score is cabin code.
        - **marketingAirline** (*String*) -- two character IATA code of the
          marketing airline that publishes and markets the flight booked
          under its own airline designator and flight number. The marketing
          airline should be displayed to travelers as the primary airline.
        - **operatingAirline** (*String*) -- two character IATA code of the
          airline operating this specific segment
        - **marketingAirlineName** (*String*) -- The name of the airline
          that publishes and markets the flight booked under its own airline
          designator and flight number
        - **operatingAirlineName** (*String*) -- The airline operating this
          specific segment
        - **flightNumber** (*String*) - the flight number for the specific
          flight, normally displayed as XXYYYY, where XX is the marketing
          airline's code, and YYYY is this number

.. _Stop:

Stop
----

    A stop is either the departure, or the arrival part of a segment.

    :JSON Parameters:
        - **dateTime** (*String*) -- time of the stop (in ISO format)
        - **airport** (*Airport*) -- location of the stop

          - **terminal** -- the relevant terminal of the airport specified
            below (this will be ``null`` is the airport has only one terminal)
          - **name** (*String*) -- official airport name of the specific stop
          - **code** -- the three letter IATA code of the airport the stop is
            at

        - **city** (*City*) -- location city name of the stop

          - **name** (*String*) -- official city name of the specific stop
          - **code** -- the three letter IATA code of the city the stop
            belongs to

.. _Comfort score:

Comfort score
-------------

    Comfort score is a variable that indicates how comfortable each
    combination option is. It is based on different aspects of the
    flight, e.g.:

     - Total time elapsed from first departure to last arrival
     - Number of flight segments (:ref:`Segment` *\[ \]*)
     - Cabin type
     - Passenger capacity of aircrafts
     - Red-eye flight status, meaning flight leaves or departs at an
       inconvenient time
     - The time elapsed between flight segments

.. _CheckInCharges:

Check-in charges
---------------

    :JSON Parameters:
        - **type** (*String*) - Usually "Airport Check-in" or "Web Check-in"
        - **currency** (*String*)
        - **amount** (*Float*)

Response Codes
==============

 - **404 'No flights available'**
 - **404 'No flight found for return leg'**
 - **404 'Search does not include a required country'** - It is possible to set
   rules to disallow search queries that don't include a specific country in the
   itinerary. If a search request doesn't match the set filter, this is returned
 - **500 'external provider rejected the request - please try again'**: This is
   the generic error sent when we receive an unknown error as response from the
   provider

Examples
========

Request
-------

    **JSON:**

    .. sourcecode:: json

        {
          "fromLocation": "BUD",
          "toLocation": "LON",
          "departureDate": "2014-05-15T00:00:00",
          "returnDate": "2014-05-20T00:00:00",
          "persons":[
            {
              "passengerType":"ADT",
              "quantity": 2
            },
            {
              "passengerType":"CHD",
              "quantity": 1
            }
          ],
          "flexible_date_search_reference": {
            "cookie": "1234567asdf",
            "extra_days": 2
          }
        }

Response
--------

    **JSON:**

    .. sourcecode:: json

        {
          "flightResultSet": [
            {
              "breakdown": [
                {
                  "passengerFare": {
                    "fare": 52.8627,
                    "tax": 21.1229,
                    "ticketDesignators": [],
                    "type": "ADT",
                    "quantity": 1,
                    "fare_in_preferred_currencies": [
                      {
                        "currency":GBP",
                        "fare": 72,
                        "tax": 21.1229,
                      },
                      {
                        "currency": "USD",
                        "fare": 66,
                        "tax": 21.1229,
                      }
                    ],
                  }
                }
              ],
              "currency": "EUR",
              "total_fare": 57.8627,
              "ticketing_fee": 5,
              "total_fare_in_preferred_currencies": [
                {
                  "currency":GBP",
                  "total_fare": 72,
                },
                {
                  "currency": "USD",
                  "total_fare": 66,
                }
              ],
              "ticketing_fee_in_preferred_currencies": [
                {
                  "currency":GBP",
                  "ticketing_fee": 3.66,
                },
                {
                  "currency": "USD",
                  "ticketing_fee": 5.74,
                }
              ],
              "price_charged_by_provider": {
                "currency":GBP",
                 "ticketing_fee": 3.66,
                 "total_fare": 72,
              },
              "combinations": [
                {
                  "bookingId": "15_0_0",
                  "comfortScore": 47,
                  "firstLeg": {
                    "elapsedTime": "0230",
                    "flightSegments": [
                      {
                        "operatingAirlineName": "British Airways",
                        "marketingAirlineName": "British Airways",
                        "aircraft": "Airbus Industries A320",
                        "arrival": {
                          "airport": {
                            "name": "Stansted",
                            "terminal": null,
                            "code": "STN"
                          },
                          "city": {
                            "code": "LON",
                            "name": "London"
                          },
                          "dateTime": "2014-06-05T23:00:00"
                        },
                        "marketingAirline": "BA",
                        "operatingAirline": "FR",
                        "departure": {
                          "airport": {
                            "terminal": null,
                            "code": "BUD"
                            "name": "Liszt Ferenc Intl",
                          },
                          "city": {
                            "code": "BUD",
                            "name": "Budapest"
                          },
                          "dateTime": "2014-06-05T21:30:00"
                        },
                        "flightNumber": "867",
                        "availableSeats": 9,
                        "cabin": "economy",
                        "class": "Y",
                      }
                    ]
                  },
                  "serviceFeeAmount": 5.0,
                  "comfortScore": 50,
                  "service_fee_in_preferred_currencies": [
                    {
                      "currency":GBP",
                      "service_fee": 3.66,
                    },
                    {
                      "currency": "USD",
                      "service_fee": 5.74,
                    }
                  ],
                }
              ]
            }
          ]
        }

.. _Flexible_Date_Search:

Flexible Date Search
--------------------

    Returns the cheapest flight option for all the possible combinations of
    the departure and arrival dates +/- the number of ``extraDays``.

    .. warning::
        To proceed with the flight workflow after a flexible date search, a
        regular search request must be sent with the parameters of the chosen
        option. It is not possible to make a booking based on booking IDs
        returned in the flexible date search response! Please include the
        **flexible_date_search_reference** parameters in the regular search sent
        after a flexible date search.

    :JSON Parameters:
        - **fromLocation** (*String*) -- departure location, given as IATA code
        - **toLocation** (*String*) -- destination, given as IATA code
        - **departureDate** (*String*) -- date of departure
        - **returnDate** (*String*) -- date of return
        - **id** (*String*) -- unique identifier of the result

Examples
--------

Request
-------

    **JSON:**

    .. sourcecode:: json

        {
          "fromLocation": "BUD",
          "toLocation": "LON",
          "departureDate": "2014-05-15T00:00:00",
          "returnDate": "2014-05-20T00:00:00",
          "persons":[
            {
              "passengerType":"ADT",
              "quantity": 2
            },
            {
              "passengerType":"CHD",
              "quantity": 1
            }
          ],
          "extraDays": 3,
        }

Response
--------

    **JSON:**

    .. sourcecode:: json

        {
          "flightResultSet": [
            {
              "flightResult": {
                "_comment": "same as in regular search response"
              },
              "fromLocation": "BUD",
              "toLocation": "LON",
              "departureDate": "2015-04-29T00:00:00Z",
              "returnDate": "2015-05-06T00:00:00Z",
              "id": "0648ae1d-3b48-4a88-b317-a5ca65fd2d67",
            }
          ]
        }

.. _Flight_Details:

---------
 Details
---------

Request
=======

.. http:get:: /flights/:booking_id

    **booking_id** is the booking ID of the :ref:`Combination` to get the
    details of

Response Body
=============

    :JSON Parameters:
        - **flightDetails** (:ref:`FlightDetailsContainer`) -- root container

.. _FlightDetailsContainer:

FlightDetails
-------------

    .. warning::
        While the ``price`` field contains the ticket's final price, baggages
        are not included in that, as the user may be able to choose from
        different baggage tiers. It is the travel site's responsibility to add
        the cost of the passenger's baggages themselves as an extra cost.

    .. note::
        Providers return prices in the travel site's preferred currency
        automatically. In the rare case that they might fail to do so, the
        Allmyles API will convert the prices to the flight fare's currency
        automatically, based on the provider's currency conversion data.

    :JSON Parameters:
        - **rulesLink** (*String*) -- link to the airline's rules page (hosted
          on the airline's website)
        - **baggageTiers** (:ref:`BaggageTier` *\[ \]*) -- contains the
          different options the passenger has for bringing baggages along. The
          book request will need to contain the ID of one of these objects in
          the baggage field.
        - **carryOnBaggageTiers** (:ref:`carryOnBaggageTier`) -- contains the
          different options of cabin baggages. The book request will need
          to contain the ID of one of these objects in the carry-on baggage
          field.
        - **fields** (:ref:`FormFields`) -- contains field validation data
        - **price** (:ref:`Price`) -- contains the final price of the ticket
          (including the credit card surcharge, but not the baggages)
        - **result** (:ref:`flight-result`) -- contains an exact copy of the
          result from the :ref:`Flight_Search` call's response
        - **options** (:ref:`FlightOptions`) -- contains whether certain
          options are enabled for this flight
        - **surcharge** (:ref:`Price`) -- contains the credit card surcharge
          for this flight
        - **price_in_preferred_currencies** (:ref:`Price` *\[ \]*) -- contains
          the final price of the ticket converted to the client's preferred
          currencies
        - **surcharge_in_preferred_currencies** (:ref:`Price` *\[ \]*) -- contains
          the credit card surcharge for this flight converted to the client's preferred
          currencies

.. _BaggageTier:

BaggageTier
-----------

    These objects define the passenger's options for taking baggages on the
    flight. Each passenger can choose one of these for themselves.

    .. note::
        Keep in mind that while the tier ID's value may seem closely related to
        the other fields, it's not guaranteed to contain any semantic meaning at
        all.

    :JSON Parameters:
        - **tier** (*String*) -- the ID for this baggage tier (this is used to
          refer to it when booking)
        - **price** (:ref:`Price`) -- contains the price of the baggage tier
        - **max_weights** (*Float* *\[ \]*) -- the maximum weight of each
          piece of baggage a passenger can take in this tier in kg, can be an
          empty array if there's no limit. Having multiple items in this array
          means that for the specified price, the passenger can check in as many
          baggages as there are items in the array. Can be an empty list if data
          is present in the *total* field.
        - **total** -- Some airlines don't limit the weights of each bag, only
          the total weight of all the bags, and the number of bags.

          - **weight** (*Float*) -- maximum summed weight of all the bags the
            passenger can take
          - **number_of_bags** (*Int*) -- number of bags that the passenger can
            take
        - **price_in_preferred_currencies** (:ref:`Price` *\[ \]*) -- contains
          the price of the baggage tier converted to the client's preferred
          currencies

.. _carryOnBaggageTier:

CarryOnBaggageTier
---------------------
    These objects define the passenger's options for taking cabin baggages
    on the flight. Each passenger can choose one of these for themselves.

    :JSON Parameters:
        - **tier** (*String*) -- the ID for this baggage tier (this is used to
          refer to it when booking)
        - **price** (:ref:`Price`) -- contains the price of the baggage tier
        - **description** (*String*) -- A basic description of the carry-on
          baggage's size, e.g. `Small cabin bag`. Exact dimensions should be
          checked on the airline's website.
        - **price_in_preferred_currencies** (:ref:`Price` *\[ \]*) -- contains
          the price of the baggage tier converted to the client's preferred
          currencies

.. _FormFields:

Form Fields
-----------

Form fields define criteria for field validation, making it easy to generate
HTML form elements.

      :JSON Parameters:
        - **passengers** (:ref:`FormField` *\[ \]*) -- contains validation
          data for Passenger fields
        - **contactInfo** (:ref:`FormField` *\[ \]*) -- contains validation
          data for Contact Info fields
        - **billingInfo** (:ref:`FormField` *\[ \]*) -- contains validation
          data for Billing Info fields

.. _FormField:

Form Field
----------

    :JSON Parameters for ``select`` fields:
        - **tag** (*String*) -- HTML tag type, in this case ``select``
        - **options** (*String [ ]*) -- value options of the field
        - **attributes** (:ref:`Attributes` *\[ \]*) -- attributes of the field

    :JSON Parameters for ``input`` fields:
        - **tag** (*String*) -- HTML tag type, in this case ``input``
        - **attributes** (:ref:`Attributes` *\[ \]*) -- attributes of the field

.. _Attributes:

Attributes
----------

    :JSON Parameters:
        - **name** (*String*) -- one of :ref:`Field_Names`
        - **data-label** (*String*) -- user friendly field label
        - **type** (*String*) -- type of input data (``†ext`` or ``email``)
        - **maxLength** (*Float*)
        - **required** (*String*) -- if present, field is required
        - **pattern** (*String*) -- regex pattern of valid data

.. _Field_Names:

Field Names
-----------

    :Passenger:
        - namePrefix
        - firstName
        - middleName
        - lastName
        - gender
        - birthDate
        - document/type
        - document/id
        - document/issueCountry
        - document/dateOfExpiry

    :Contact and Billing Info:
        - name
        - email
        - address/addressLine1
        - address/addressLine2
        - address/addressLine3
        - address/cityName
        - address/zipCode
        - address/countryCode
        - phone/countryCode
        - phone/areaCode
        - phone/phoneNumber

.. _Price:

Price
-----

    :JSON Parameters:
        - **amount** (*Float*) -- the amount of money in the currency below
        - **currency** (*String*) -- the currency of the amount specified, can
          be null when the amount is zero

.. _FlightOptions:

FlightOptions
-------------

    **{optionName}** below refers to the following names:

        - seatSelectionAvailable
        - travelfusionPrepayAvailable

    :JSON Parameters:
        - **{optionName}** (*Boolean*) -- whether the option is enabled or not

Response Codes
==============

 - **404 'search first'**
 - **412 'a request is already being processed'**: This error comes up even
   when the other request is asynchronous (i.e. when we are still processing a
   search request). The response for async requests does not need to be
   retrieved for this error to clear, just wait a few seconds.
 - **412 'request is not for the latest search'**: One case where this error
   is returned is when a customer is using multiple tabs and trying to select
   a flight from an old result list.

Examples
========

Response
--------

    **JSON:**

    .. sourcecode:: json

        {
          "flightDetails": {
            "rulesLink": null,
            "baggageTiers": [
                {
                    "tier": "0",
                    "price": {
                        "currency": null,
                        "amount": 0.0
                    },
                    "max_weights": [],
                    'total': {
                        'weight': None,
                        'number_of_bags': None,
                    },
                    "price_in_preferred_currencies": [
                      {
                        "currency":GBP",
                        "amount": 0.0
                      },
                      {
                        "currency": "USD",
                        "amount": 0.0
                      }
                    ],
                },
                {
                    "tier": "1",
                    "price": {
                        "currency": "HUF",
                        "amount": 15427.0
                    },
                    "max_weights": [15.0],
                    'total': {
                        'weight': None,
                        'number_of_bags': None,
                    },
                    "price_in_preferred_currencies": [
                      {
                        "currency":GBP",
                        "amount": 10.0
                      },
                      {
                        "currency": "USD",
                        "amount": 12.0
                      }
                    ],
                },
                {
                    "tier": "2",
                    "price": {
                        "currency": "HUF",
                        "amount": 37024.8
                    },
                    "max_weights": [],
                    'total': {
                        'weight': 45,
                        'number_of_bags': 2,
                    },
                    "price_in_preferred_currencies": [
                      {
                        "currency":GBP",
                        "amount": 20.0
                      },
                      {
                        "currency": "USD",
                        "amount": 22.0
                      }
                    ],
                }
            ],
            "carryOnBaggageTiers": [
                {
                    "tier": "1",
                    "price": {
                        "currency": "null",
                        "amount": 0.0
                    },
                    "description": "Small cabin bag",
                },
                {
                    "tier": "2",
                    "price": {
                        "currency": "HUF",
                        "amount": 8000.0
                    },
                    "description": "Large cabin bag",
                },
                "price_in_preferred_currencies": [
                  {
                    "currency":GBP",
                    "amount": 20.0
                  },
                  {
                    "currency": "USD",
                    "amount": 22.0
                  }
                ],
            ],
            "fields": {
              "passengers": [
                {
                  "tag": "select",
                  "options": ["Mr", "Ms", "Mrs"],
                  "attributes": [
                    {
                      "key": "required",
                      "value": "required"
                    },
                    {
                      "key": "name",
                      "value": "persons/0/namePrefix"
                    },
                    {
                      "key": "data-label",
                      "value": "Name Prefix"
                    }
                  ],
                },
              ],
              "contact_info": [
                {
                  "tag": "input",
                  "attributes": [
                    {
                      "key": "maxLength",
                      "value": "30"
                    },
                    {
                      "key": "type",
                      "value": "text"
                    },
                    {
                     "key": "name",
                     "value": "billingInfo/name"
                    },
                    {
                      "key": "data-label",
                      "value": "Name"
                    }
                  ],
                },
              ],
              "billing_info": [
                {
                  "_comment": "trimmed in example for brevity's sake"
                },
              ]
            },
            "price": {
              "currency": "EUR",
              "amount": 4464.46
            },
            "result": {
              "_comment": "trimmed in example for brevity's sake"
            },
            "options": {
              "seatSelectionAvailable": false,
              "travelfusionPrepayAvailable": false
            },
            "surcharge": {
              "currency": "EUR",
              "amount": 5.0
              "card_type": "CA",
            },
            "price_in_preferred_currencies": [
              {
                "currency":GBP",
                "amount": 3269
              },
              {
                "currency": "USD",
                "amount": 5162
              }
            ],
            "surcharge_in_preferred_currencies": [
              {
                "currency":GBP",
                "amount": 5.0
                "card_type": "CA",
              },
              {
                "currency": "USD",
                "amount": 5.0
                "card_type": "CA",
              }
            ],
          }
        }

.. _Flight_Booking:

---------
 Booking
---------

    .. note::
        When booking LCC flights, there are two possible scenarios.
        By *default*, the Allmyles API does not send the book request to the
        external provider until the ticketing call arrives, so there's no
        response---an HTTP 204 No Content status code is returned.
        If you have chosen *alternative* providers (you have to contact the Allmyles
        support about this first), the booking flow of LCC flights is very similar to
        that of traditional flights. In this case the book response differs just a bit
        from the traditional book response - please refer to the book response
        specifications for detailed information.

Request
=======

.. http:post:: /books

    :JSON Parameters:
        - **bookBasket** (*String*) -- the booking ID of the :ref:`Combination`
          to book
        - **billingInfo** (:ref:`Flight_Billing`) -- billing info for ticket creation
        - **contactInfo** (:ref:`Flight_Contact`) -- contact info for ticket creation
        - **persons** (:ref:`Passenger` *\[ \]*) -- the list of passengers
        - **userData** (:ref:`User_Data`) -- information about the end user
        - **tenantReferenceId** (*String*) -- *(optional)* ID of the booking on the tenant's
          side - can be useful for debugging purposes

.. _Flight_Contact:

Contact
-------

    :JSON Parameters:
        - **address** (:ref:`Flight_Address`) -- address of the the contact person
        - **email** (*String*) -- email of the contact person
        - **firstName** (*String*)
        - **middleName** (*String*) -- *(optional)* submission of this parameter is mandatory if the person in question has a middle name
        - **lastName** (*String*) 
        - **phone** (:ref:`Flight_Phone`) -- phone number of the contact person
        
.. _Flight_Billing:

Billing
-------

    :JSON Parameters:
        - **address** (:ref:`Flight_Address`) -- address of the entity in question
        - **email** (*String*) -- email of the entity in question
        - **firstName** (*String*) -- name of the entity in question, if the entity is an organization this is the only name field that is required
        - **middleName** (*String*) -- *(optional)* submission of this parameter is mandatory if the person in question has a middle name and must not be sent in if the entity in question is an organization
        - **lastName** (*String*) -- *(optional)* submission of this parameter is mandatory if the entity in question is a person and it must not be included if the entity is an organization
        - **phone** (:ref:`Flight_Phone`) -- phone number of the entity in question

.. _Flight_Address:

Address
-------

    :JSON Parameters:
        - **addressLine1** (*String*)
        - **addressLine2** (*String*) -- *(optional)*
        - **addressLine3** (*String*) -- *(optional)*
        - **cityName** (*String*)
        - **zipCode** (*String*)
        - **countryCode** (*String*) -- the two letter code of the country

.. _Flight_Phone:

Phone
-----

    :JSON Parameters:
        - **countryCode** (*String*)
        - **areaCode** (*String*)
        - **phoneNumber** (*String*) - Must be at least 7 characters long.

.. _Passenger:

Passenger
---------

    :JSON Parameters:
        - **birthDate** (*String*) -- format is ``YYYY-MM-DD``
        - **document** (:ref:`FlightDocument`) -- data about the identifying
          document the passenger wishes to travel with
        - **email** (*String*)
        - **namePrefix** (*String*) -- one of ``Mr``, ``Ms``, or ``Mrs``
        - **firstName** (*String*)
        - **middleName** (*String*) -- *(optional)*
        - **lastName** (*String*)
        - **gender** (*String*) -- one of ``MALE`` or ``FEMALE``
        - **passengerTypeCode** (*String*) -- one of :ref:`PassengerTypes`
        - **baggage** (*String*) -- one of the tier IDs returned in the
          flight details response
        - **carryOnBaggage** (*String*) -- one of the tier IDs returned
          in the flight details response

.. _FlightDocument:

Document
--------

    :JSON Parameters:
        - **id** (*String*) -- document's ID number
        - **dateOfExpiry** (*String*) -- format is YYYY-MM-DD
        - **issueCountry** (*String*) -- two letter code of issuing country
        - **type** (*String*) -- one of :ref:`DocumentTypes`

Response Body
=============

    .. note::
        Again: **by default, there's no response body for LCC book requests!**
        An HTTP 204 No Content status code confirms that Allmyles saved the
        sent data for later use.

    .. warning::
        If you have chosen alternative providers - that means there IS a book response
        for LCC flights, **this is the response that contains the exact final price** that
        should be shown to the traveler. This price contains the baggage and hand luggage
        surcharges, if applicable.

    .. warning::
        The format of :ref:`Flight_Contact` and :ref:`flight-result` objects contained
        within this response might slightly differ from what's described in
        this documentation as requested. This will be fixed in a later version.

    :JSON Parameters:
        - **price** (:ref:`Price`) -- final price updated with baggage surcharges.
          **Only in alternative LCC book response!**
        - **pnr** (*String*) -- the PNR locator which identifies this booking
        - **lastTicketingDate** (*String*) -- the timestamp of when it's last
          possible to create a ticket for the booking, in ISO format
        - **bookingReferenceId** (*String*) -- the ID of the workflow at
          Allmyles; this is not currently required anywhere later, but can be
          useful for debugging
        - **contactInfo** (:ref:`Flight_Contact`) -- contains a copy of the data
          received in the :ref:`Flight_Booking` call
        - **flightData** (:ref:`flight-result`) -- contains a copy of the
          result from the :ref:`Flight_Search` call's response

Response Codes
==============

 - **303 'Unable to book this flight - please select a different bookingId'**:
   This error is returned when the external provider encounters a problem such
   as a discrepancy between actual flight data and what they returned from
   their cache before. This happens very rarely, or never in production.
 - **404 'search first'**
 - **412 'a request is already being processed'**: This error comes up even
   when the other request is asynchronous (i.e. when we are still processing a
   search request). The response for async requests does not need to be
   retrieved for this error to clear, just wait a few seconds.
 - **412 'Already booked.'**: This denotes that either us or the external
   provider has detected a possible duplicate booking, and has broken the flow
   to avoid dupe payments.
 - **412 'already booked'**: This is technically the same as the error above,
   but is encountered at a different point in the flow. The error messages are
   only temporarily not the same for these two errors.
 - **412 'request is not for the latest search'**
 - **500 'could not book flight'**: This is the generic error returned when we
   encounter an unknown/empty response from the external provider
 - **504 'external gateway timed out - book request might very well have been
   successful!'**: The booking might, or might not have been completed in this
   case. The flow should be stopped, and the customer should be contacted to
   complete the booking.
 - **504 'Could not retrieve virtual credit card, flight not booked. An IRN
   should be sent to payment provider now.'**

Examples
========

Request
-------

    **JSON:**

    .. sourcecode:: json

        {
          "bookBasket": ["1_0_0"],
          "billingInfo": {
            "address": {
              "addressLine1": "Váci út 13-14",
              "cityName": "Budapest",
              "countryCode": "HU",
              "zipCode": "1234"
            },
            "email": "ccc@gmail.com",
            "name": "Kovacs Gyula",
            "phone": {
              "areaCode": "30",
              "countryCode": "36",
              "phoneNumber": "1234567"
            }
          },
          "contactInfo": {
            "address": {
              "addressLine1": "Váci út 13-14",
              "cityName": "Budapest",
              "countryCode": "HU",
              "zipCode": "1234"
            },
            "email": "bbb@gmail.com",
            "name": "Kovacs Lajos",
            "phone": {
              "areaCode": "30",
              "countryCode": "36",
              "phoneNumber": "1234567"
            }
          },
          "persons": [
            {
              "baggage": "0",
              "carryOnBaggage": "1",
              "birthDate": "1974-04-03",
              "document": {
                "dateOfExpiry": "2016-09-03",
                "id": "12345678",
                "issueCountry": "HU",
                "type": "Passport"
              },
              "email": "aaa@gmail.com",
              "firstName": "Janos",
              "gender": "MALE",
              "lastName": "Kovacs",
              "namePrefix": "Mr",
              "passengerTypeCode": "ADT"
            }
          ]
        }

Response
--------

    **JSON:**

    .. sourcecode:: json

        {
          "bookingReferenceId": "req-cfd7963b187a4fe99702c0373c89cb16",
          "contactInfo": {
            "address": {
              "city": "Budapest",
              "countryCode": "HU",
              "line1": "Madach ut 13-14",
              "line2": null,
              "line3": null
            },
            "email": "testy@gmail.com",
            "name": "Kovacs Lajos",
            "phone": {
              "areaCode": "30",
              "countryCode": "36",
              "number": "1234567"
            }
          },
          "flightData": {
            "_comment": "trimmed in example for brevity's sake"
          },
          "lastTicketingDate": "2014-05-16T23:59:59Z",
          "pnr": "6YESST"
        }

.. _Flight_Payment:

---------
 Payment
---------

This is where Allmyles gets the payment data.

Allmyles is a payment platform agnostic solution. When we receive a
transaction ID that points to a successful payment by the passenger, we
essentially take that money from any Payment Service Provider (PSP),
and forward it to the provider to buy a ticket in the :ref:`Flight_Ticketing` step.

Request
=======

.. http:post:: /payment

    :JSON Parameters:
        - **paymentId** (*String*) -- the transaction ID identifying the
          successful transaction at your PSP
        - **basket** (*String[ ]*) -- the booking IDs the payment is for

Response Body
=============

    **N/A:**

    Returns an HTTP 204 No Content status code if successful.

Response Codes
==============

 - **412 'a request is already being processed'**: This error comes up even
   when the other request is asynchronous (i.e. when we are still processing a
   search request). The response for async requests does not need to be
   retrieved for this error to clear, just wait a few seconds.
 - **412 'book request should have been received'**

Examples
========

Request
-------

    **JSON:**

    .. sourcecode:: json

        {
          "paymentId": "12345678",
          "basket": ["2_1_0"]
        }

.. _Flight_Ticketing:

-----------
 Ticketing
-----------

Two important notes:

1. Call this only when the passenger's payment completely went through! (That
   is, after the payment provider's IPN has arrived, confirming that the
   transaction did not get caught by the fraud protection filter.)
2. After this call has been made **do not issue refunds** unless the Allmyles
   API explicitly tells you to. It's way better to just correct ticketing
   errors manually than to fire automatic refunds even if the ticket purchase
   might already be locked in for some reason.

Request
=======

.. http:get:: /tickets/:booking_id

    **booking_id** is the booking ID of the :ref:`Combination` to create a
    ticket for

Response Body
=============

    By default, this is just an abstraction for the book call when buying an
    LCC ticket (there's no separate book and ticketing calls for those flights).
    This means the response differs greatly depending on whether the flight is
    traditional or LCC booked through the *default* providers.

    If you have chosen *alternative* providers (you would have to contact the
    Allmyles support about this first), there **is** a separate book response for
    LCC flights, but the ticket response is the same as described below.

    :JSON Parameters for traditional flights:
        - **tickets** (*Ticket [ ]*) -- the purchased tickets

          - **passenger** (*String*) -- the name of the passenger the ticket
            was purchased for
          - **passenger_type** (*String*) -- one of :ref:`PassengerTypes`
          - **ticket** (*String*) -- the ticket number which allows the
            passenger to actually board the plane
          - **price** (*TicketPrice*)

            - **currency** (*String*)
            - **total_fare** (*Float*) -- The total amount of money the
              passenger paid for his ticket, including tax.
            - **tax** (*Float*) -- The total amount of tax the passenger had to
              pay for this ticket.
          - **baggage**

            - **quantity** (*Int*) -- The maximum quantity of baggage the
              passenger can bring along
            - **unit** (*String*) -- Units of measurement
          - **price_in_preferred_currencies** (*TicketPrice [ ]*) -- the
            ticket price converted to the client's preferred currencies
            - **currency** (*String*)
            - **total_fare** (*Float*)
            - **tax** (*Float*)
        - **flightData** (:ref:`flight-result`) -- contains a copy of the
          result from the :ref:`Flight_Search` call's response
        - **contactInfo** (:ref:`Flight_Contact`) -- contains a copy of the data
          received in the :ref:`Flight_Booking` call

    :JSON Parameters for LCC flights:
        - **ticket** (*String*) -- the ticket number (LCC PNR) for this booking
        - **pnr** (*String*) -- the PNR locator which identifies this booking
        - **bookingReferenceId** (*String*) -- the ID of the workflow at
          Allmyles; this is not currently required anywhere later, but can be
          useful for debugging
        - **contactInfo** (:ref:`Flight_Contact`) -- contains a copy of the data
          received in the :ref:`Flight_Booking` call
        - **flightData** (:ref:`flight-result`) -- contains a copy of the
          result from the :ref:`Flight_Search` call's response
        - **baggageTiers** (:ref:`BaggageTier` *\[ \]*) -- the baggage tier
          option the passenger has chosen
        - **carryOnBaggageTiers** (:ref:`carryOnBaggageTier` *\[ \]*) -- the
          carry-on baggage tier option the passenger has chosen


Response Codes
==============

In case of errors (referring to response code 202 and 5xx), the client is
expected to either have a correct the ticketing manually, or send periodic
:ref:`Flight_Ticketing_Status` requests until a definitive response is given
(one of the following statuses: 'successful', 'failed', or 'unknown'.) This
should take no longer than 40 minutes. Tickets with an unknown status still
require manual intervention.

 - **202 'Warning: e-ticket could not be issued due to technical difficulties.
   Please contact youragent.'**: When this error occurs, the actual ticket is
   purchased, but an unknown error happens later on in the flow.
 - **412 'a request is already being processed'**: This error comes up even
   when the other request is asynchronous (i.e. when we are still processing a
   search request). The response for async requests does not need to be
   retrieved for this error to clear, just wait a few seconds.
 - **412 'no payment data given'**
 - **412 'book request should have been received'**
 - **412 'book response should have been received'**
 - **500 'booking failed, cannot create ticket'**: This error is returned if
   the book response we last received from the provider contained an error.
 - **503 'error while creating ticket - please try again later'**: This is the
   generic error we return when receiving an unknown response for the ticket
   request. No refund should be sent without manually checking if the ticket
   has been issued first.
 - **504 'ticket creation timed out - but could very well have been
   successful!'**: Almost the same as above, refunds are definitely not safe in
   this case.

Examples
========

Response
--------

    **JSON for traditional flights:**

    .. sourcecode:: json

        "body": {
          "tickets": [
            {
              "passenger": "Mr Janos kovcas",
              "passenger_type": "ADT",
              "ticket": "125-4838843038",
              "price": {
                "currency": "HUF",
                "total_fare": 26000.0,
                "tax": 17800.0
              }
              "baggage": {
                "quantity": 1,
                "unit": "PC",
              },
              "price_in_preferred_currencies": [
              {
                "currency":GBP",
                "total_fare": 60.48,
                "tax": 41.41
              },
              {
                "currency": "USD",
                "total_fare": 94.84,
                "tax": 64.93
              }
            ],
            },
            {
              "passenger": "Mr Janos kascvo",
              "passenger_type": "ADT",
              "ticket": "125-4838843039",
              "price": {
                "currency": "HUF",
                "total_fare": 26000.0,
                "tax": 17800.0
              }
              "baggage": {
                "quantity": 1,
                "unit": "PC",
              },
              "price_in_preferred_currencies": [
              {
                "currency":GBP",
                "total_fare": 60.48,
                "tax": 41.41
              },
              {
                "currency": "USD",
                "total_fare": 94.84,
                "tax": 64.93
              }
            ],
            }
          ],
          "flightData": {
            "_comment": "trimmed in example for brevity's sake"
          },
          "contactInfo": {
            "address": {
              "city": "Budapest",
              "countryCode": "HU",
              "line1": "Madach ut 13-14",
              "line2": null,
              "line3": null
            },
            "email": "testytesty@gmail.com",
            "name": "Kovacs Lajos",
            "phone": {
              "areaCode": "30",
              "countryCode": "36",
              "number": "1234567"
            }
          }
        }

    **JSON for LCC flights:**

    .. sourcecode:: json

        {
          "bookingReferenceId": "req-d65c00dc43ba4ad798e5478803575aab",
          "contactInfo": {
            "address": {
              "city": "Budapest",
              "countryCode": "HU",
              "line1": "Madach ut 13-14",
              "line2": null,
              "line3": null
            },
            "email": "testytesty@gmail.com",
            "name": "Kovacs Lajos",
            "phone": {
              "areaCode": "30",
              "countryCode": "36",
              "number": "1234567"
            }
          },
          "flightData": {
            "_comment": "trimmed in example for brevity's sake"
          },
          "lastTicketingDate": null,
          "pnr": "6YE2LM",
          "ticket": "0XN4GTO",
          "baggageTiers": {
            "tier": "2",
            "max_weights": [15.0, 20.0],
            "price": {
              "amount": 37024.8,
              "currency": HUF
            },
            "price_in_preferred_currencies": [
              {
                "currency":GBP",
                "amount": 10.0
              },
              {
                "currency": "USD",
                "amount": 12.0
              }
            ],
          },
          "carryOnBaggageTiers": {
            "tier": "2",
            "description": "Large cabin bag",
            "price": {
              "amount": 8000.0,
              "currency": HUF
            },
            "price_in_preferred_currencies": [
              {
                "currency":GBP",
                "amount": 10.0
              },
              {
                "currency": "USD",
                "amount": 12.0
              }
            ],
          }
        }

.. _Flight_Ticketing_Status:

------------------
 Ticketing Status
------------------

This call enables checking the result of a ticketing request. This is useful
when it's unclear whether the ticketing process went through, due to a failure
at external providers, in Allmyles' systems, on the client's server, or anywhere
in between. The request will identify the correct workflow based on the cookie
header's contents, which must match whatever was sent in the ticket request.

If you're using **alternative providers** and an LCC booking returns with the
status **pending** or **unknown**, keep in mind that the ticket could still be
created successfully in the next 72 hours. You should keep making periodic
:ref:`Flight_Ticketing_Status` requests at a reduced rated until a **successful**
or **failed** status is returned or the 72-hour period is over.

The periodic checks should be made at most once every 5 minutes.

Available statuses
==================

 - **inactive**: this is the status returned when the ticketing process has not
   been initiated yet, i.e. before a :ref:`Flight_Ticketing` request is
   sent
 - **pending**: the ticket creation is still in progress
 - **successful**: the ticket has been successfully created. PNR data will be
   passed alongside this status, including the ticket number(s).
 - **failed**: the ticket creation failed, and the fare can be refunded (do
   note that this is the only status in which refunds can be automatically made)
 - **unknown**: it is not possible to programmatically determine the outcome of
   the request. The passenger's money should be held until a human identifies
   the issue and determines whether the ticket exists or not.

Request
=======

.. http:get:: /tickets/:booking_id/status

    **booking_id** is the booking ID of the :ref:`Combination` whose ticket's
    status we are interested in

Response Body
=============


    :JSON Parameters:
        - **status** (*String*) -- one of the statuses
        - **pnr** (:ref:`PNR <pnr-data>`) -- the pnr object that a
          :ref:`Get_Booking` request would return about the flight --- this
          includes the ticket number(s) as well

Examples
========

Response
--------

    **JSON for traditional flights:**

    .. sourcecode:: json

        {
            "status": "successful",
            "pnr": {
                "deleted": false,
                "id": "3L4TMN",
                "passengers": [
                    {
                        "birth_date": "1974-01-01",
                        "email": "test@example.com",
                        "name": "SMFDETH HYRASESN/MR",
                        "traditional_ticket": "125-5249156160",
                        "type": "ADT"
                    },
                    {
                        "birth_date": "1974-01-01",
                        "email": null,
                        "name": "SMIATTASDH OSAJOEONHTDNHO/MR",
                        "traditional_ticket": "125-5249156161",
                        "type": "ADT"
                    }
                ]
            }
        }


.. _Flight_Rules:

-------
 Rules
-------

This call returns the terms and conditions of the flight in question, or a link
to them if the raw text isn't available (in case of LCC flights).

Request
=======

.. http:get:: /flights/:booking_id/rules

    **booking_id** is the booking ID of the :ref:`Combination` to get the
    rules of

Response Body
=============

    :JSON Parameters:
        - **rulesResultSet** (*RulesResultSet*) -- root container

          - **rules** (:ref:`Rule` *\[ \]*) -- contains the flight rule texts,
            is returned only for traditional flights
          - **link** (*String*) -- contains a link to the airline's rules
            page, is returned only for LCC flights

.. _Rule:

Rule
----

    :JSON Parameters:
        - **code** (*String*) - the machine readable identifier code for the
          given section in the rules
        - **title** (*String*) - the human readable section title for the block
        - **text** (*String*) - the section's raw rule text body

Response Codes
==============

 - **404 'search first'**
 - **412 'a request is already being processed'**: This error comes up even
   when the other request is asynchronous (i.e. when we are still processing a
   search request). The response for async requests does not need to be
   retrieved for this error to clear, just wait a few seconds.
 - **409 'request is not for the latest search'**

Examples
========

Response
--------

    **JSON (for LCC):**

    .. sourcecode:: json

        {
          "rulesResultSet": {
            "link": "https://www.ryanair.com/en/terms-and-conditions"
          }
        }

    **JSON (for traditional):**

    .. sourcecode:: json

        {
          "rulesResultSet": {
            "rules": [
              {
                "code": "OD",
                "text": "NONE UNLESS OTHERWISE SPECIFIED",
                "title": "OTHER DISCOUNTS"
              },
              {
                "code": "SO",
                "text": "STOPOVERS NOT PERMITTED ON THE FARE COMPONENT.",
                "title": "STOPOVERS"
              },
            ]
          }
        }

.. _Get_Booking:

-------------
 Get Booking
-------------

This call returns the details of a booking identified by a PNR locator.
This makes it possible to re-open an expired session and send a ticketing
request based on the PNR locator after the initial session is closed.

Request
=======

.. http:get:: /books/:pnr_locator

    **pnr_locator** is a unique identifier of the booking, received
    at the book response.

.. _pnr-data:

Response Body
=============

    :JSON Parameters:
        - **pnr** (*pnr*) -- root container

          - **passengers** (*Passenger [ ]*) -- the list of
            passengers

            - **birth_date** (*String*) -- format is ``YYYY-MM-DD``
            - **traditional_ticket** (*String*) - the ticket number which allows
              the passenger to actually board the plane (or ``null`` if flight
              is LCC)
            - **type** (*String*) -- one of :ref:`PassengerTypes`
            - **email** (*String*)
            - **name** (*String*) -- the name of the passenger the booking was
              made for
          - **id** (*String*) -- the PNR locator which identifies the
            booking
          - **lcc_ticket** (*String*) -- the ticket number which allows
            the passenger to actually board the plane
            (or ``null`` if flight is traditional)


Response Codes
==============

 - **404 'PNR not found'**
 - **403 'PNR belongs to another auth token'**

Examples
========

Response
--------

    **JSON:**

    .. sourcecode:: json

        {
          "pnr": {
            "passengers": [
              {
                "birth_date": "1974-01-01",
                "traditional_ticket": "123-5249155974",
                "type": "ADT",
                "email": "test@gmail.com",
                "name": "KOVACS JANOS/MR"
              }
            ],
            "id": "3KWQUK",
            "lcc_ticket": null
          }
        }

.. _Cancel:

---------------
 Cancel Booking
---------------

This call cancels the booking identified in the request. Bookings can only
be cancelled before a ticket is created. **Only bookings of traditional
flights can be cancelled!**

Request
=======

.. http:delete:: /books/:pnr_locator

    **pnr_locator** is a unique identifier of the booking, received
    at the book response.

Response Body
=============

    **N/A:**

    Returns an HTTP 204 No Content status code if successful.

Response Codes
==============

 - **403 'PNR belongs to another auth token'**
 - **404 'PNR not found'**
 - **409 'Booking already cancelled.'**
 - **409 'Booked flights can only be cancelled before ticket is created.'**
