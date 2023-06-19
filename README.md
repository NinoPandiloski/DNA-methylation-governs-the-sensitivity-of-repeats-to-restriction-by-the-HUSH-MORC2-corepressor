# DNA-methylation-status-governs-repeat-restriction-by-the-HUSH-MORC2-complex

## Heatmaps:
All heatmaps were created using DeepTools' computeMatrix and plotHeatmap. An example of how they were created is as follows:
```
computeMatrix scale-regions -m 6000 -p 4 -S <BigWig signal> -R <BED coordinates of genomic regions> -a 1000 -b 1000 -o <matrix_file>
plotHeatmap -m <matrix_file> --sortRegions descend --colorMap "Reds" -o <heatmap>.pdf
```


## Peak Calling using SEACR:
Prior to peak calling, we performed pre-processing of the BAM files as suggested by the creators of SEACR on their GitHub page (https://github.com/FredHutch/SEACR):
  ```
        bedtools bamtobed -bedpe -i <input_BAM> > <output_BED>
        awk '$1==$4 && $6-$2 < 1000 <<print $0>>' <output_BED> > <output_clean_BED>
        cut -f 1,2,6 <output_clean_BED> | sort -k1,1 -k2,2n -k3,3n > <output_fragments_BED>
        bedtools genomecov -bg -i <output_fragments_BED> -g <genome_index> > <output_BED_BEDGRAPH>
  ```

Once the BEDgraph files were created, we could start peak calling:
```
bash SEACR_1.3.sh <experimental_BEDGRAPH> <Control_BEDGRAPH> norm stringent <PeakCalling_output>.stringent.bed
```


## Peak Calling using HOMER:
To establish a set of baseline peaks we performed the following steps:
1. Created tagDirectories using HOMER's makeTagDirectory:
```
makeTagDirectory <output_tag_dir> <input_BAMs>
```

2. Called peaks with HOMER's findPeaks:
```
findPeaks <control_H3K9me3_signal_tagDir> -style histone -o <output_baseline_peaks>.txt -i <control_IgG_signal_tagDir>
```

3. H3K9me3 peaks were filtered according to length, where all peaks shorter than 1kb were filtered out. Additionally, all non-canonical chromosomes were filtered out as well.

4. Called peaks specifically gained in the knockdown by repeating step 1 with <HUSHi_H3K9me3_signal_tagDir> as the first input.

5. Finally, both the baseline peaks and knockdown specific peaks were concatenated and merged in order to obtain an extensive peak list.

## Differential enrichment over peaks:
- To get the differential enrichment over the called peaks, we employed getDifferentialPeaks from HOMER in both directions - using the control as the background and the knockdown as the target, and vice versa.
```
1. getDifferentialPeaks <output_merged_peaks>.txt <LacZ_H3K9me3_tagDir> <HUSHi_H3K9me3_tagDir> -F 0
                                              &&
2. getDifferentialPeaks <output_merged_peaks>.txt <HUSHi_H3K9me3_tagDir> <LacZ_H3K9me3_tagDir> -F 0
```
3. Steps 1 and 2 were repeated by adding the flag "-same", in order to gain the peaks which are not significantly enriched in neither direction.

4. All the peaks were sorted according to direction in which they show enrichment - lost in CRISPRi, not significant or gained in CRISPRi.

5. Finally, the pValues and fold changes were extracted to create a volcanoPlot.
