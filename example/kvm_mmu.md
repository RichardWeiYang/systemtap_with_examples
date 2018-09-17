Finally, I reached my original purpose to learn systemtap.

> Investigate how kvm mmu system works

Now let systemtap to show its power.

# Dump level of page in kvm_mmu_pages

To utilize memory efficiently, kvm would zay unsync children. It will traverse
the SP and form a kvm_mmu_pages to record potential unsync children.

Actually, the level of those pages in kvm_mmu_pages has a patten. Now let's
use systemtap to prove it.

Here it the systemtap script

```
probe module("kvm").function("mmu_pages_first")
{
        if ($pvec->nr <= 4)
                next;
        printf("kvm_mmu_pages level (%s)\n", pp());
        printf("SP Index:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15\n");
        printf("SP Level:");
        for (i = 0; i < $pvec->nr; i++) {
                printf("%3d", $pvec->page[i]->sp->role->level);
        }
        printf("\n");
        exit();
}
```

Then is the result

```
kvm_mmu_pages level (module("kvm").function("mmu_pages_first@arch/x86/kvm/mmu.c:2264"))
SP Index:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15
SP Level:  4  3  2  1  1  3  2  1  1  3  2  1
```

Do you find something in the output?

Every time, it traverses to the leaf (level 1) and then go to another subtree.
To be simple, this is a depth first traverse.
