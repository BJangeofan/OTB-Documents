# Auto-generate reference documentation in build directory
if(DEFINED OTB_INSTALL_PREFIX)
  set(OTB_PREFIX ${OTB_INSTALL_PREFIX})
else()
  set(OTB_PREFIX ${OTB_DIR})
endif()

CONFIGURE_FILE(${CookBook_SOURCE_DIR}/CMake/RunApplicationsDocGenerator.sh.cmake.in
               ${CMAKE_CURRENT_BINARY_DIR}/RunApplicationsDocGenerator.sh
               @ONLY
               )

ADD_CUSTOM_TARGET(AppRefDocTex
                  ALL
                  COMMAND bash ${CMAKE_CURRENT_BINARY_DIR}/RunApplicationsDocGenerator.sh
                  COMMENT "Auto-generating Application Reference Documentation"
                  )

# Find part files
FILE(GLOB  RELATED_SRCS RELATIVE "${CookBook_SOURCE_DIR}/Latex" "*")

# Add the main latex doc
ADD_LATEX_DOCUMENT(CookBook.tex INPUTS ${RELATED_SRCS} MANGLE_TARGET_NAMES DEFAULT_PDF IMAGE_DIRS ../Art ../Art/MonteverdiImages ../Art/QtImages ../Art/SARImages USE_INDEX)

ADD_DEPENDENCIES(CookBook_pdf AppRefDocTex)

FIND_PROGRAM(TEX4HT_COMPILER
  NAMES htlatex
  PATHS ${MIKTEX_BINARY_PATH}
        /usr/bin
)

IF(NOT TEX4HT_COMPILER)
  MESSAGE("htlatex executable was not found")
ENDIF(NOT TEX4HT_COMPILER)


# Option for html generation
SET(TEX4HT_OPTIONS html,4,subsubsections+)
SET (HTML_OUTPUT "CookBook-${OTB_VERSION_MAJOR}.${OTB_VERSION_MINOR}")

IF(LATEX_COMPILER)
  SET(ITK_TEXINPUTS ${CookBook_SOURCE_DIR}/../Latex:${CookBook_SOURCE_DIR}/Latex:${CookBook_SOURCE_DIR}/Art:${CookBook_BINARY_DIR}/Art:${CookBook_BINARY_DIR}/Latex)
  IF (TEX4HT_COMPILER)
    CONFIGURE_FILE(${CookBook_SOURCE_DIR}/Tex4htWrapper.sh.in
      ${CookBook_BINARY_DIR}/Tex4htWrapper.sh)
    SET (TEX4HT_WRAPPER1 ${CookBook_BINARY_DIR}/Tex4htWrapper.sh)
    SET (TEX4HT_WRAPPER2 "")
    ADD_CUSTOM_TARGET(CookBookHTML
                      ALL
                      COMMAND sh ${CookBook_BINARY_DIR}/Tex4htWrapper.sh CookBook
                      WORKING_DIRECTORY ${CookBook_BINARY_DIR}
                      COMMENT "Building HTML documentation"
                      )
    ADD_DEPENDENCIES(CookBookHTML CookBook_pdf)
  ENDIF (TEX4HT_COMPILER)
ENDIF(LATEX_COMPILER)

# Rename the CookBook.pdf file into CookBook-version.pdf
SET(FINAL_DOC_NAME "${LATEX_TARGET}-${OTB_VERSION_MAJOR}.${OTB_VERSION_MINOR}.${OTB_VERSION_PATCH}")
ADD_CUSTOM_TARGET(CookBookRenamePdf
                  ALL
   		  ${CMAKE_COMMAND} -E rename ${LATEX_TARGET}.pdf ${FINAL_DOC_NAME}.pdf
                  COMMENT "Renaming the pdf document" 
       )
ADD_DEPENDENCIES(CookBookRenamePdf CookBook_pdf)


