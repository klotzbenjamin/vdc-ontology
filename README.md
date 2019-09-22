# VDC: a Vehicle Signals and Attribute Ontology
This folder contains the research project carried out around the development, extension and usage of VDC. 
VDC is an ontology created from the combination of the SOSA patterns for observations and the Event ontology, applied to driving situations.

## Documentation
This repository contains the html documentation about VDC, automatically generated using WIDOCO.
The rendered page is available at http://automotive.eurecom.fr/vdc

## Competency questions for VSSo
Here is a list of competency questions that served to evaluate VDC, expressed when possible as SPARQL queries on VDC datasets.

### Single domain states

#### Who is the driver  currently?
```SPARQL
SELECT ?driver ?name
WHERE {
?driver a foaf:Person;
	foaf:name ?name;
	vdc:hasRole vdc:Driver;
	vdc:hasMentalState ?state.
?observation sosa:hasObservedPropery ?state;
	sosa:hasPhenomenonTime ?time.
}ORDER BY DESC (?time)
LIMIT 1
```

#### What is the traffic flow around this car?
```SPARQL
SELECT ?flow
WHERE {
	?signal a datex:TrafficFlow, vdc:RoadState.
	?observation a sosa:Observation;
		sosa:hasObservedProperty ?signal;
		sosa:hasSimpleResult ?flow;
		sosa:hasPhenomenonTime ?time.
}ORDER BY DESC (?time)
LIMIT 1
```

#### What are the driver informations?
```SPARQL
SELECT ?driver ?property ?value
WHERE {
	
	{ 
     	SELECT ?driver {
      		?driver a foaf:Person;
				vdc:hasRole vdc:Driver;
				vdc:hasMentalState ?state.
			?observation sosa:hasObservedPropery ?state;
				sosa:hasPhenomenonTime ?time.
	    }ORDER BY DESC (?time)
		LIMIT 1
	}

	?driver ?property ?value.
}
```


#### What type of road is the car on?
```SPARQL
SELECT ?label ?type
WHERE {
	?segment a datacron:TrajectoryPart, vdc:Road, ?type;
		rdfs:label ?label.

	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }

	FILTER { ?type != datacron:TrajectoryPart }
}
```

#### What is the current speed limit where my car is located?
```SPARQL
SELECT ?speedLimit
WHERE {
	?segment a datacron:TrajectoryPart, vdc:Road.

	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }

	?signal a tti:SpeedLimit, vdc:RoadState.

	?observation a sosa:Observation;
		sosa:hasObservedProperty ?signal;
		sosa:hasSimpleResult ?speedLimit;
		geo:location ?segment.
}
```

#### What is the temperature, precipitation and weather description in my area? (With the weather ontology)
```SPARQL
PRFIX wo: <https://www.auto.tuwien.ac.at/downloads/thinkhome/ontology/WeatherOntology.owl>
SELECT ?temperature ?precipitation ?description
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }

	?temperatureSignal a wo:Temperature, vdc:WeatherState.
	?precipitationSignal a wo:Precipitation, vdc:WeatherState.
	?descriptionSignal a wo:WeatherCondition, vdc:WeatherState.

	?temperatureObservation a sosa:Observation;
		sosa:hasObservedProperty ?temperatureSignal;
		sosa:hasSimpleResult ?temperature;
		geo:location ?segment.
	?precipitationObservation a sosa:Observation;
		sosa:hasObservedProperty ?precipitationSignal;
		sosa:hasSimpleResult ?precipitation;
		geo:location ?segment.
	?descriptionObservation a sosa:Observation;
		sosa:hasObservedProperty ?descriptionSignal;
		sosa:hasSimpleResult ?description;
		geo:location ?segment.
}
```

#### Has the driver been aggressive?
```SPARQL
SELECT ?isAggressive
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }

	?driver a foaf:Person;
		dco:hasRole dco:Driver;
		dco:hasBehavioralState ?state.


	{ 
     	SELECT count(?observation) AS ?count {
      		?state a mfoem:000015.
			?observation sosa:hasObservedProperty ?state;
				sosa:hasResult ?behavior.
				sosa:hasPhenomenonTime ?time.
	    }
	}

	BIND( IF(?count>0,"yes","no") as ?isAggressive )

}
```
#### When was the last time the driver was bored?
```SPARQL
SELECT ?time
WHERE {
	?driver a foaf:Person;
		dco:hasRole dco:Driver;
		dco:hasMentalState ?state.
	?state a mfoem:000166.
	?observation sosa:hasObservedProperty ?state;
		sosa:hasPhenomenonTime ?time.

}ORDER BY DESC (?time)
LIMIT 1
```
#### What are the sensors in this car ?
```SPARQL
SELECT ?sensor
WHERE {
	?sensor sosa:observes ?signal.
	?signal a vsso:ObservableSignal.
}
```
#### What is the dimensions of this car?
```SPARQL
SELECT ?length ?width ?height
WHERE { ?branch vsso:length ?length;
	vsso:width ?width;
	vsso:height ?height.
}
```

#### What is this place?
```SPARQL
SELECT ?label
WHERE {
	?segment a datacron:TrajectoryPart, vdc:Road;
		datacron:hasRegion ?region.
	?region datacron:associatedWith ?label.

	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }
}
```
#### What type of place is it?
```SPARQL
SELECT ?type
WHERE {
	?segment a datacron:TrajectoryPart, vdc:Road;
		datacron:hasRegion ?region.
	?region a ?type.

	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }
}
```

#### What is the mental state of this passenger?
```SPARQL
SELECT ?stateType
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }

	?passenger a foaf:Person;
		dco:hasRole dco:Passenger;
		dco:hasMentalState ?state.

	?state a ?stateType.

	?observation sosa:hasObservedProperty ?state;
		geo:location ?segment.

}
```

#### What is the origin POI of my trajectory?
```SPARQL
SELECT ?label ?latitude ?longitude
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyFollows ?nextSegment }

	?segment datacron:hasStartingPoint ?point.
	?point datacron:associatedWith ?label;
		datacron:hasLatitude ?latitude;
		datacron:hasLongitude ?longitude.
}
```







### Events in single domains

#### In which events is currently involved the driver?
```SPARQL
SELECT ?event
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyFollows ?nextSegment }

	?event a vdc:Event, event:Event;
		event:agent :Driver;
		event:place ?segment.
}
```

#### What are the objects around my car?
Inexpressable in the current VDC model.

#### What are the risks around my car currently?
```SPARQL
SELECT ?risk
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyFollows ?nextSegment }

	?risk a vdc:Event, hazard:Exposure;
		event:agent :Vehicle.
}
```

#### What is this hazard involving the driver?
```SPARQL
SELECT ?type
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyFollows ?nextSegment }

	?event a vdc:Event, hazard:Exposure, ?type;
		event:agent :Driver;
		event:place ?segment.

	FILTER(?type != vdc:Event)
	FILTER(?type != hazard:Exposure)
}
```

#### Who is involved in this hazard?
```SPARQL
SELECT ?agent
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyFollows ?nextSegment }

	?event a vdc:Event, hazard:Exposure, ?type;
		event:agent :Driver, ?agent.

	?agent a foaf:Person.

}
```















### States combinations from multiple domains

#### Am I overspeeding?
```SPARQL
SELECT ?overspeeding
WHERE {
	?segment a datacron:TrajectoryPart.
	FILTER NOT EXISTS { ?segment datacron:directlyPrecedes ?nextSegment }

	?signal a tti:SpeedLimit.
	?observation a sosa:Observation;
		sosa:hasObservedProperty ?signal;
		sosa:hasSimpleResult ?speedLimit;
		geo:location ?segment.
	{
		SELECT ?speed
		WHERE {
			?signal a vsso:VehicleSpeed.
			?observation a sosa:Observation;
				sosa:hasObservedProperty ?signal;
				sosa:hasSimpleResult ?speed;
				sosa:hasPhenomenonTime ?time.
		}ORDER BY DESC (?time)
		LIMIT 1
	}
	

	BIND( IF(?speed>?speedLimit,"yes","no") as ?overspeeding )
}
````

#### Does the trajectory go through the city of Munich?
Not implemented yet.

#### What is the most common mental state of the driver on the current trajectory?
```SPARQL
SELECT ?stateType
WHERE {
	:Trajectory datacron:hasPart ?segment.
	?segment a datacron:TrajectoryPart.

	?driver a foaf:Person;
		dco:hasRole dco:Driver;
		dco:hasMentalState ?state.

	{
		SELECT ?stateType COUNT(?observation) AS ?count
		WHERE {
			?state a ?stateType.
			?observation sosa:hasObservedProperty ?state;
				geo:location ?segment.
		}

	}
}ORDER BY DESC (?count)
LIMIT 1
```
#### What is the next POI I will have within a range of 500m on my path?
Not yet implemented

#### On which side of the road is the next POI I will have within a range of 500m on my path?
Not yet implemented



### Cross-domain events

#### When/Where did this event occur?
```SPARQL
SELECT ?location ?time
WHERE {
	?event a vdc:Event;
		event:time ?time;
		event:place ?location.
}
```
#### What the cause of the current congestion?
```SPARQL
SELECT ?
WHERE {
	{
		SELECT ?disruption
		WHERE {
			?disruption a td:DisruptiveEvent, vdc:Event;
			event:time ?time.
		}ORDER BY DESC (?time)
		LIMIT 1
	}

	?disruption event:hasFactor ?factor.
}
```

#### How many tti:TurnLeft maneuvers have the driver done the last hour?
```SPARQL
SELECT COUNT (DISTINCT ?turn) AS ?turns
WHERE {
	?turn a tti:TurnLeft, vdc:Event;
		event:agent :Driver.
		event:time ?time.

	BIND( (NOW() - "P0DT1H"^^xsd:dayTimeDuration) AS ?beginTime ) .
    FILTER( ?time >= ?beginTime )
}
```

#### What is the value of the gear signal and place at this given time?
```SPARQL
SELECT ?value ?latitude ?longitude
WHERE {
	?gear a vsso:CurrentGear.
	?observation a sosa:Observation;
		sosa:observedProperty ?gear;
		sosa:hasSimpleResult ?result;
		geo:lat ?latitude;
		geo:long ?longitude;
		sosa:phenomenonTime :time.
}
```