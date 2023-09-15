# dds-datamodels-oaris

This datamodel is an enhanced version of the OMG® OARIS™ 2.0 beta 2datamodel
that you can find here: https://www.omg.org/spec/OARIS/2.0/Beta2/About-OARIS

## Repo Organization

### Versioning & Branches

This repository stores the different versions of the OARIS datamodel in
different branches. Additionally, it contains `enhanced` versions of the
original datamodel. This enhanced versions modifies the original datamodel
including the latest IDL features and other potential improvements. The
different changes are explained in their own readme file.

The branches in this repo follow this pattern:

 - main: this contains the latest enhanced version
 - version/x.y\[-(version_specifier\[-enhanced\]\]

For example, `version/2.0-beta-enhanced`

The `version_specifier` is added if a non-final version is being used. This
information appears in the datamodel website.

The `-enhanced` indicates that it contains the enhanced version of the specified
datamodel version.

### Folders

This repository contains one folder called `datamodel` that contains the
representation of this datamodel. Internally, that folder contains the different
files that implement the datamodel. It must contain an `idl` folder that
includes the IDL files of the datamodel. Additionally, other folders with the
name of the technology used for the representation of the datamodel may be
present. For example: `xml`, `json`...

## Changes on the Datamodel

This enhanced version contains several changes in the datamodel for the
OARIS version 2.0 beta2:

 - Replaced the comments in the IDL files with a custom annotation `@doc("")`.
 - Replaced the key definition with the IDL 4.2 `@key` annotation.
 - Replaced the unions with only one case (TRUE) to represent optional members
  with the IDL 4.2 `@optional` annotation.

Optional elements example:
```
    // a simple union type, to represent an optional value
    union illumination_request_frequency_channel_type switch (boolean)
    {
        // the value when present
        case TRUE : frequency_channel_type value;
    };
    ...
    struct illumination_request_type
    {
        track_id_type target_track_id;
        illumination_request_own_missile_track_id_type own_missile_track_id;
        org::omg::c4i::Domain_Model::Common_Types::Coordinates_and_Positions::absolute_duration_type illumination_period;
        illumination_request_frequency_channel_type frequency_channel;
        org::omg::c4i::Domain_Model::Common_Types::anonymous_blob_type additional_parameters;
    };
```

This is replaced by:
```
    struct illumination_request_type
    {
        track_id_type target_track_id;
        illumination_request_own_missile_track_id_type own_missile_track_id;
        org::omg::c4i::Domain_Model::Common_Types::Coordinates_and_Positions::absolute_duration_type illumination_period;
        @optional frequency_channel_type frequency_channel;
        org::omg::c4i::Domain_Model::Common_Types::anonymous_blob_type additional_parameters;
    };
```

## Testing
In order to test this datamodel after the applied changes, `rtiddsgen` from
RTI Connext 6.1.2 has been used. A convenient script has been used to run this
automatically. You can find the script here: https://github.com/angelrti/run-rtiddsgen

In order to run the script in all files the following command was called:
```
run-rtiddsgen -v -D datamodel/idl -o delete_generated_files --additional-options="-verbosity 1" --dds-type
```
