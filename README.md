# Geoembeddings Convention Metadata

- **UUID**: 61c12cc5-0e28-4056-999a-480cf3fb7e4c
- **Name**: geoemb:
- **Schema URL**: "https://raw.githubusercontent.com/geo-embeddings/embeddings-zarr-convention/refs/tags/v1/schema.json"
- **Spec URL**: "https://github.com/geo-embeddings/embeddings-zarr-convention/blob/v1/README.md"
- **Scope**: Group
- **Extension Maturity Classification**: Proposal
- **Owner**: @geo-embeddings

## Description

This convention defines metadata for geospatial embedding groups stored in Zarr format. It provides standardized attributes for describing embedding provenance, including the encoder model, source data, processing parameters, and quality metrics. All properties use the `geoemb:` namespace prefix and are placed at the root `attributes` level following the [Zarr Conventions Specification](https://github.com/zarr-conventions/zarr-conventions-spec).

The convention supports two embedding types:

- **Pixel embeddings**: Per-pixel dense embeddings where each spatial location has an embedding vector
- **Patch (Chip) embeddings**: Image patch (chip) embeddings where non-overlapping or overlapping regions are encoded into single vectors

This convention is designed to be compatible with [GeoZarr conventions](https://geozarr.org/conventions.html) and can be used alongside:

- `proj:` for CRS information
- `spatial:` for coordinate transforms and bounding boxes

- Examples:
  - [Minimal pixel embedding](examples/minimal_example.json)
  - [Full chip embedding with all fields](examples/full_example.json)

## Motivation

- **Reproducibility**: Track model provenance and inference parameters for scientific reproducibility
- **Interoperability**: Standard metadata enables embedding search and comparison across datasets
- **Discoverability**: Structured metadata supports catalog queries and dataset discovery
- **Quality transparency**: Uncertainty fields promote responsible AI practices

## Convention Registration

The convention must be registered in `zarr_conventions`:

```json
{
  "zarr_conventions": [
    {
      "schema_url": "https://raw.githubusercontent.com/geo-embeddings/embeddings-zarr-convention/refs/tags/v1/schema.json",
      "spec_url": "https://github.com/geo-embeddings/embeddings-zarr-convention/blob/v1/README.md",
      "uuid": "61c12cc5-0e28-4056-999a-480cf3fb7e4c",
      "name": "geoemb:",
      "description": "Geoembeddings convention for geospatial embedding arrays with model provenance"
    }
  ]
}
```

## Applicable To

This convention can be used with these parts of the Zarr hierarchy:

- [x] Group
- [ ] Array

## Properties

All properties are placed at the root `attributes` level with the `geoemb:` prefix.

### Required Fields

| Field Name         | Type              | Description                                          |
| ------------------ | ----------------- | ---------------------------------------------------- |
| geoemb:type        | "pixel" \| "chip" | **REQUIRED**. Type of embedding                      |
| geoemb:dimensions  | integer           | **REQUIRED**. Dimensionality of the embedding vector |
| geoemb:model       | string (URL)      | **REQUIRED**. Reference to the encoder model         |
| geoemb:source_data | string (URL)      | **REQUIRED**. Reference to the source dataset        |
| geoemb:data_type   | string            | **REQUIRED**. Data type of stored embeddings (e.g., "float32", "int8") |

**Note**: When `geoemb:type` is `"chip"`, the `geoemb:chip_layout` field is also required.

### Optional Fields

| Field Name                 | Type                                        | Description                                        |
| -------------------------- | ------------------------------------------- | -------------------------------------------------- |
| geoemb:gsd                 | number                                      | Ground sample distance in meters                   |
| geoemb:chip_layout         | [Chip Layout Object](#chip-layout-object)   | Chip layout configuration (required for chip-type) |
| geoemb:quantization        | [Quantization Object](#quantization-object) | Compression/quantization details                   |
| geoemb:benchmark           | \[string]                                   | URLs to benchmark evaluation results               |

### Example (Minimal Pixel Embedding)

```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      { "name": "geoemb:", "uuid": "61c12cc5-0e28-4056-999a-480cf3fb7e4c" }
    ],
    "geoemb:type": "pixel",
    "geoemb:dimensions": 768,
    "geoemb:model": "https://huggingface.co/made-with-clay/Clay",
    "geoemb:source_data": "https://registry.opendata.aws/sentinel-2-l2a-cogs/",
    "geoemb:data_type": "float32"
  }
}
```

### Example (Full Chip Embedding)

```json
{
  "zarr_format": 3,
  "node_type": "group",
  "attributes": {
    "zarr_conventions": [
      { "name": "geoemb:", "uuid": "61c12cc5-0e28-4056-999a-480cf3fb7e4c" }
    ],
    "geoemb:type": "chip",
    "geoemb:dimensions": 768,
    "geoemb:model": "https://huggingface.co/made-with-clay/Clay",
    "geoemb:source_data": "https://registry.opendata.aws/sentinel-2-l2a-cogs/",
    "geoemb:data_type": "float32",
    "geoemb:gsd": 10.0,
    "geoemb:chip_layout": {
      "layout_type": "regular_grid",
      "chip_size": [256, 256],
      "stride": [256, 256]
    }
  }
}
```

## Complex Objects

### Chip Layout Object

Configuration for chip-type embeddings describing how the source imagery was divided.

| Field Name      | Type                          | Description                                             |
| --------------- | ----------------------------- | ------------------------------------------------------- |
| layout_type     | "regular_grid" \| "irregular" | **REQUIRED**. Type of chip layout                       |
| chip_size       | \[integer, integer]           | **REQUIRED**. Chip dimensions [height, width] in pixels |
| stride          | \[integer, integer]           | Stride between chips [y, x]. Defaults to chip_size      |
| grid_id         | string                        | Identifier for a predefined grid system                 |
| grid_definition | string (URL)                  | URL to grid definition document                         |

### Quantization Object

Details for embeddings that have been quantized for compression.

| Field Name      | Type         | Description                                                                          |
| --------------- | ------------ | ------------------------------------------------------------------------------------ |
| method          | string       | **REQUIRED**. Quantization method (e.g., "linear", "product_quantization", "binary") |
| original_dtype  | string       | **REQUIRED**. Original data type before quantization (e.g., "float32")               |
| quantized_dtype | string       | Data type after quantization (e.g., "int8")                                          |
| scale           | number       | Scale factor for linear dequantization                                               |
| offset          | number       | Offset for linear dequantization                                                     |
| link            | string (URL) | URL to quantization codebook or lookup table                                         |

## Known Implementations

_If you implement or use this convention, please add your implementation to this list by submitting a pull request._

## Acknowledgements

This convention is based on the [embeddings-stac-specification](https://github.com/geo-embeddings/embeddings-stac-specification) and follows the [Zarr Conventions](https://github.com/zarr-conventions) template.
