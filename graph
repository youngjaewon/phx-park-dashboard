graph TD
    %% 스타일 정의
    classDef input fill:#f9f,stroke:#333,stroke-width:2px,color:black;
    classDef decision fill:#fffac8,stroke:#d4a017,stroke-width:2px,color:black;
    classDef process fill:#e1f5fe,stroke:#0277bd,stroke-width:2px,color:black;
    classDef output fill:#c8e6c9,stroke:#388e3c,stroke-width:2px,color:black;
    classDef finalOutput fill:#dcedc8,stroke:#558b2f,stroke-width:4px,color:black;

    %% 시작점
    START([START]) --> InputPolygons[/Input: Park Polygons/]:::input

    %% Decision 1
    InputPolygons --> D1{Decision 1:<br/>Do Survey123 or Public<br/>points exist?}:::decision
    D1 -- Yes --> SetSourceD1[Set source = 'survey' or 'public']:::process
    SetSourceD1 --> ToStepC((Go to Step C<br/>Input))

    D1 -- No --> D2{Decision 2:<br/>Does OSM contain a<br/>Parking Lot inside?}:::decision

    %% Decision 2
    D2 -- Yes --> CreateParkingPoint[Create access point from<br/>parking lot centroid/boundary]:::process
    CreateParkingPoint --> SetSourceD2[Set source = 'osm_parking']:::process
    SetSourceD2 --> ToStepC

    D2 -- No --> D3{Decision 3:<br/>Does boundary intersect<br/>OSM Roads?}:::decision

    %% Decision 3 - Yes Branch (Intersection)
    D3 -- Yes --> CreateIntersection[Create points at<br/>boundary-road intersections]:::process
    CreateIntersection --> SetSourceD3Y[Set source = 'boundary_intersection']:::process
    SetSourceD3Y --> RoadClass[Road classification & labeling]:::process
    RoadClass --> AssignAccessType[Assign access_type:<br/>'driving' or 'pedestrian'<br/>based on ORS rules]:::process
    AssignAccessType --> MergePoints

    %% Decision 3 - No Branch (Snap)
    D3 -- No --> SnapRoad[Snap to nearest road<br/>and create point]:::process
    SnapRoad --> SetSourceD3N[Set source = 'snapped']:::process
    SetSourceD3N --> QAQC[QA/QC for snapped points]:::process
    QAQC --> ComputeSnapDist[Compute snap_distance]:::process
    ComputeSnapDist --> FlagIssues["Flag issues:<br/>qc_flag = 'large_snap_distance', etc."]:::process
    FlagIssues --> MergePoints

    %% Step B Output Consolidation
    MergePoints((Convergence)) --> StepBOutput[/"Step B Output:<br/>Access Points Table<br/>Fields: park_id, geometry, source,<br/>access_type, snap_distance, qc_flag"/]:::output
    StepBOutput --> StepCInput

    %% Step C Process
    subgraph Step_C_Process [Step C: Service Area Generation]
        ToStepC --> StepCInput[/"Input: Park Access Points"/]:::input
        StepCInput --> FilterPoints["Filter: Select 'driving' points<br/>(Optional: exclude bad qc_flag)"]:::process
        FilterPoints --> ORSCall["Call OpenRouteService Isochrone API:<br/>Profile: driving-car<br/>Range: 10 minutes"]:::process
        ORSCall --> FinalOut
    end

    %% 최종 출력
    FinalOut[/"Output: 10-minute driving<br/>service areas polygons"/]:::finalOutput
