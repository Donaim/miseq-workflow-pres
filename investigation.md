
# Bad cases

---

Sample `HIV0913NFL-PLT14H24-NFLHIVDNA_S83`.

Stitcher plots: 

main iva:
file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/241868/stitcher_plot.svg

referenceless stitcher iva:
file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/240597/stitcher_plot.svg

referenceless stitcher haploflow:
file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/240082/stitcher_plot.svg

We see that the best result is actually with Haploflow.

The referenceless stitcher iva version looks the worst.
The reason for this is that a good contig got covered by an anomaly contig.
But the anomaly contig is recognized by BLAST, and bot recognized by mappy.

Investigation results at `investigation/HIV0913NFL-PLT14H24-NFLHIVDNA_S83/`.

---

Sample `HIV0913NFL-PLT12C23-NFLHIVDNA_S44`.


Stitcher plots:

main iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/240558/stitcher_plot.svg


referenceless stitcher iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/241831/stitcher_plot.svg

The stitcher version is worse.
The reason for it is very similar to that of sample `HIV0913NFL-PLT14H24-NFLHIVDNA_S83`.

---

Sample `POSITION15-HIV-AD_S31`.

main iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/242295/stitcher_plot.svg

referenceless stitcher iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/241024/stitcher_plot.svg

The no-stitcher version aligns much better to the reference.
Notice how the subtype is different.
But the blast match is much, much better!

The underlying problem seems to be in the fact that the referenceless stitcher could not split a bad contig.

---

Sample `73098A-HCV_S42`.

main iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/242171/stitcher_plot.svg

referenceless stitcher iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/240900/stitcher_plot.svg

This is a weird example.
The concordance score is identical value.
But alignment score is worse for referenceless stitcher iva.

Conclusion: this is just a comparison error.

---

Sample `57368A-11-HLA-B-63594A-V3-2-V3LOOP_S11`.

main iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/241940/stitcher_plot.svg

referenceless stitcher iva:

file:///media/mybtrfs/home-submodule/my-link-files/root/home/user1/.local/share/miyka/root/repositories/uoept29crvyclcar/wd/home/my/project/other/haploflow-evaluation-2/megacompare/runs/240669/stitcher_plot.svg

This error is not reproducible on `v7.17.0-1264-g7de1a5421`.

---

Sample `57368A-1-HLA-B-63455A-V3-1-V3LOOP_S1`.

Same deal as with `57368A-11-HLA-B-63594A-V3-2-V3LOOP_S11`.

---

Sample `57368A-7-HLA-B-63529A-V3-1-V3LOOP_S7`

Same deal as with `57368A-11-HLA-B-63594A-V3-2-V3LOOP_S11`.

---

Sample `E102398-V3-1-V3LOOP_S55`.

The only contig is just missing in the stitcher version.
This is some kind of bug not related to the stitcher.

---

Sample `63902A-V3-3-V3LOOP_S39`.




---
