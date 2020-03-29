## Determine Event Overlaps
An implementation agnostic approach to determine if 2 events overlap.

---

![Allen's 13 Basic relations]({{ site.baseurl }}/assets/images/allens_13_basic_relations.png)

### Shorthand Version
For any given 2 events: `event1`, `event2`, they overlap if below condition is `true`
```
event1.start_time <= event2.end_time and event1.end_time >= event2.start_time
```
The `Rails Guides Date Range Overlap` link in References section below explains beautifully how we arrive this condition. Please follow the link for a detailed explaination.
Next, I have shared few examples based on different databases.


### Examples

#### Postgres:
Consider an events table as follows:

uuid | start_time | end_time | duration_in_seconds
--- | --- | ---
123 | 2020-03-28 01:00:00 +0000 | 2020-03-28 01:00:15 +0000 | 15


Then, for an incoming event `incoming_event`, we can determine overlap by:
```ruby
def overlap?(incoming_event)
	Event.exists?(["start_time <= ? and end_time >= ?",
		incoming_event.end_time, incoming_event.start_time])
end
```
---

#### AWS DynamoDB:
Consider an events table as follows:

uuid | start_timestamp | end_timestamp | duration_in_seconds
--- | --- | ---
123 | 1585357200 | 1585357215 | 15


Then, for an incoming event `incoming_event`, we can determine overlap by:
```ruby
def self.overlap?(incoming_event)
	# set limit for number of events we need to check.
	record_limit = 50
	# Step 1: fetch persisted events started before incoming event end_timestamp:
	#    	  (event1.start_time <= event2.end_time)
	events_started_before_incoming_event = Event.where('start_timestamp.lte': incoming_event.end_timestamp)
	    .scan_index_forward(false)
	    .record_limit(record_limit)
	    .all
	# Step 2: evaluate remainder condition:
	#         (event1.end_time >= event2.start_time)
	events_started_before_incoming_event.each do |existing_event|
		  # Overlap found
	      return true if (existing_event.end_timestamp >= incoming_event.start_timestamp)
	    end
	# No Overlap found
	return false
end
```


### References
- [Allen's Interval Allegbra](https://www.ics.uci.edu/~alspaugh/cls/shr/allen.html)
- [Rails Guides Date Range Overlap](https://railsguides.net/date-ranges-overlap/)
