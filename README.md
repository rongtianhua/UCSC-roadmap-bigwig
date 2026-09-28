# UCSC-roadmap-bigwig

Region-subset Roadmap Epigenomics bigWigs served from `raw.githubusercontent.com` for use as
**UCSC Genome Browser custom tracks** and as igv.js / 3D Genome Browser local tracks.

## Why raw and not a CDN

UCSC's bbi reader issues fixed **16,384-byte HTTP Range reads and does not decompress the
response**. jsDelivr returns `content-encoding: gzip` on `.bigwig`, so UCSC receives a short
body and aborts the track. `raw.githubusercontent.com` returns the bytes verbatim.

Verify a file before trusting it:

```sh
curl -s -D- -o /dev/null -H 'Range: bytes=417792-433791' \
     -H 'Accept-Encoding: gzip, deflate' \
     https://raw.githubusercontent.com/rongtianhua/UCSC-roadmap-bigwig/master/wide/ERP27_E100_H3K27ac.bigwig
# want: HTTP/2 206, content-length: 16000, and NO content-encoding header
```

## Naming rule

Filenames use **underscores** (`ERP27_E100_H3K27ac.bigwig`). The dash spelling
(`E100-H3K27ac`) is what egg2 uses; dash-named files here 404 on every host.

## Layout

```
wide/<GENE>_<EID>_<ASSAY>.bigwig
```

`EID` = Roadmap E-code. Tissue mapping used by the PD + sarcopenia figure set:

| EID | Tissue | Colour |
|-----|--------|--------|
| E073 | DLPFC (brain) | blue |
| E074 | Substantia Nigra (brain) | light blue |
| E100 | Psoas (muscle) | dark red |
| E107 | Skeletal muscle, male | orange |
| E108 | Skeletal muscle, female | red |

`ASSAY` = `H3K27ac` | `H3K4me3` | `DNase`.

## Adding a region

```sh
# 1. slice the full egg2 bigWig down to the display window
python3 _explore_genome_browser/_scripts/region_to_bigwig.py <bed_dir> <out_dir> "chr12:14000000-15900000"

# 2. upload through the REST API (git push is unreliable on this network)
python3 _explore_genome_browser/_scripts/push_gh_api.py rongtianhua/UCSC-roadmap-bigwig <out_dir> master wide
```

Give every region at least +-1 Mb of margin around the gene so the track still fills the frame
after a zoom or a coordinate change.

## Provenance

Source: Roadmap Epigenomics hg38 imputed signal, `egg2.wustl.edu/roadmap/data/byFileType/signal/
consolidatedImputed/<assay>/<EID>-<assay>.imputed.pval.signal.hg38.bigwig`. Sliced and
re-compressed locally; no values altered.
