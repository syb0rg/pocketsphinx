SET(SOURCES	acmod.c
            allphone_search.c
            bin_mdef.c
            blkarray_list.c
            dict.c
            dict2pid.c
            fsg_history.c
            fsg_lextree.c
            fsg_search.c
            hmm.c
            kws_detections.c
            kws_search.c
            mdef.c
            ms_gauden.c
            ms_mgau.c
            ms_senone.c
            ngram_search.c
            ngram_search_fwdflat.c
            ngram_search_fwdtree.c
            phone_loop_search.c
            pocketsphinx.c
            ps_alignment.c
            ps_lattice.c
            ps_mllr.c
            ptm_mgau.c
            s2_semi_mgau.c
            state_align_search.c
            tmat.c
            vector.c
    )

add_library(libpocketsphinx OBJECT ${SOURCES})
set_property(TARGET libpocketsphinx PROPERTY POSITION_INDEPENDENT_CODE TRUE)

if (DEPENDENCIES)
	ADD_DEPENDENCIES(libpocketsphinx ${DEPENDENCIES})
endif ()