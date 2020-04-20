# /IWFND/ES_MGW_DEST_FINDER
Sample code for enhancement spot /IWFND/ES_MGW_DEST_FINDER

Recently, I had to roll out a productive SAP Fiori App in our Gateway Server to a new backend system. Besides all the necessary customizing the Odata service connection had to be enhanced. The first picture shows the initial setup.
![Table1](https://github.com/simplesm/ABAP_BD_MGW_DEST_FINDER/blob/master/img/table1.jpg)
In his [blog post](https://blogs.sap.com/2015/01/29/support-of-multiple-backend-systems-how-to-use-multi-origin-composition-and-routing/#) Andre Fischer shows in detail how we can maintain multiple backend systems for one Odata service by maintaining user roles or host names.

Assuming that this customizing works like other SAP Standard tables I thought it would be sufficient to add the new System Alias with a role assigned instead of investigating the corresponding role for the existing one.

This is how my attempt looked like: 
![Table2](https://github.com/simplesm/ABAP_BD_MGW_DEST_FINDER/blob/master/img/table2.jpg)
Unfortunately this didn't work. During my analysis I found the enhancement spot /IWFND/ES_MGW_DEST_FINDER including the Business AddIn /IWFND/BD_MGW_DEST_FINDER which we can apply to solve this problem. To activate this Business AddIn, we first need to implement the enhancement spot via SE18. The Business AddIn has a filter we must restrict to our service.
![Table2](https://github.com/simplesm/ABAP_BD_MGW_DEST_FINDER/blob/master/img/filter.jpg)
We can now place our own logic in the implementing class method /IWFND/IF_MGW_DEST_FINDER_BADI~GET_SYSTEM_ALIASES to determine the right backend system and flag it as default.

        TRY .
            DATA(lo_destin_finder) = /iwfnd/cl_destin_finder=>get_destination_finder( ).
            DATA(lt_user_roles) = lo_destin_finder->get_roles_for_user( iv_user = iv_user ).
            LOOP AT ct_system_aliases ASSIGNING FIELD-SYMBOL(<fs_alias>)
              WHERE user_role IS NOT INITIAL.
              IF line_exists( lt_user_roles[ user_role = <fs_alias>-user_role ] ).
                <fs_alias>-is_def_for_meta = abap_true.
                <fs_alias>-is_default = abap_true.
                DATA(l_role) = <fs_alias>-user_role.
              ELSE.
                CONTINUE.
              ENDIF.
            ENDLOOP.
            IF l_role IS NOT INITIAL.
              DELETE ct_system_aliases WHERE user_role <> l_role.
            ENDIF.
          CATCH cx_root.
            RAISE EXCEPTION TYPE /iwfnd/cx_mgw_dest_finder.
        ENDTRY.

The above code checks if the current user has a maintained role. In case of a maintained role the system will apply the new connection. In any other case the connection without a role.

I hope this example on how to use the Badi /IWFND/BD_MGW_DEST_FINDER helpful.

Happy coding!

