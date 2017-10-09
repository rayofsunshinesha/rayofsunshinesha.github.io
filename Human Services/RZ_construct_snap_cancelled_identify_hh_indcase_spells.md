# Before we start
{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "*logic on join snap_cancelled_earnings_redacted with hh_spell\n",
    "since there are several spells with one ch_dpa_caseid\n",
    "therefore we need to find the closeast spell from canceled date*\n",
    "\n",
    "SELECT cancel.ch_dpa_caseid, cancel.transaction_date,\n",
    "spell.recptno, spell.benefit_type, spell.end_date , \n",
    "(cancel.transaction_date-spell.end_date) as days_between_end_cancel,\n",
    "FIRST_VALUE(spell.end_date) \n",
    "OVER (PARTITION BY cancel.transaction_date \n",
    "ORDER BY ABS(cancel.transaction_date-spell.end_date))\n",
    "FROM idhs.snap_cancelled_earnings_redacted cancel\n",
    "LEFT JOIN idhs.hh_indcase_spells spell\n",
    "ON cancel.ch_dpa_caseid=spell.ch_dpa_caseid\n",
    "WHERE cancel.ch_dpa_caseid IS NOT NULL"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "* then restrict to the says_between in less than 31 days\n",
    "SELECT *, 1 as tag_cancel\n",
    "FROM(\n",
    "previous table\n",
    ")\n",
    "WHERE ABS(days_between_end_cancel)<=31\n",
    "ORDER BY ABS(days_between_end_cancel)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "* final code to generate the table  *\n",
    "SQL: new\n",
    "//tag the specific spell that canceled due to earning\n",
    "\n",
    "DROP TABLE IF EXISTS c6.snap_cancelled_identify_hh_indcase_spells ;\n",
    "CREATE TABLE c6.snap_cancelled_identify_hh_indcase_spells AS \n",
    "SELECT *, 1 as tag_cancel\n",
    "FROM(\n",
    "SELECT cancel.ch_dpa_caseid, cancel.transaction_date,\n",
    "spell.recptno, spell.benefit_type, spell.end_date , \n",
    "(cancel.transaction_date-spell.end_date) as days_between_end_cancel,\n",
    "FIRST_VALUE(spell.end_date) \n",
    "OVER (PARTITION BY cancel.transaction_date \n",
    "ORDER BY ABS(cancel.transaction_date-spell.end_date))\n",
    "FROM idhs.snap_cancelled_earnings_redacted cancel\n",
    "LEFT JOIN idhs.hh_indcase_spells spell\n",
    "ON cancel.ch_dpa_caseid=spell.ch_dpa_caseid\n",
    "WHERE cancel.ch_dpa_caseid IS NOT NULL\n",
    "AND spell.benefit_type='%foodstamp%'\n",
    ") temp\n",
    "WHERE ABS(days_between_end_cancel)<=31\n",
    "ORDER BY ABS(days_between_end_cancel)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "//tag the specific spell that canceled due to earning for foodstamp\n",
    "SQL:\n",
    "DROP TABLE IF EXISTS c6.snap_cancelled_identify_hh_indcase_spells ;\n",
    "CREATE TABLE c6.snap_cancelled_identify_hh_indcase_spells AS \n",
    "SELECT *, 1 as tag_cancel\n",
    "FROM(\n",
    "SELECT cancel.ch_dpa_caseid, cancel.transaction_date,\n",
    "spell.recptno, spell.benefit_type, spell.end_date , \n",
    "(cancel.transaction_date-spell.end_date) as days_between_end_cancel,\n",
    "FIRST_VALUE(spell.end_date) \n",
    "OVER (PARTITION BY cancel.transaction_date \n",
    "ORDER BY ABS(cancel.transaction_date-spell.end_date))\n",
    "FROM idhs.snap_cancelled_earnings_redacted cancel\n",
    "LEFT JOIN idhs.hh_indcase_spells spell\n",
    "ON cancel.ch_dpa_caseid=spell.ch_dpa_caseid\n",
    "WHERE (cancel.ch_dpa_caseid IS NOT NULL) AND \n",
    "(spell.benefit_type LIKE '%foodstamp%')\n",
    ") temp\n",
    "WHERE ABS(days_between_end_cancel)<=60\n",
    "ORDER BY ABS(days_between_end_cancel)\n",
    "\n",
    "Idea behind:\n",
    "use all cases in idhs.snap_cancelled_earnings_redacted (where ch_dpa_caseid is not missing )\n",
    "go merge with idhs.hh_indcase_spells spell to identify the specicfic spell that is canceled due to earning. \n",
    "tag the record of the smallest gap days btw transaction_date & spell.end_date \n",
    "if the gap is more than 31 days, I make the judgement that the spell is not related to the transaction_date."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {
    "collapsed": true
   },
   "outputs": [],
   "source": [
    "//tag the specific spell that canceled due to earning for tanf\n",
    "SQL:\n",
    "DROP TABLE IF EXISTS c6.tanf_cancelled_identify_hh_indcase_spells ;\n",
    "CREATE TABLE c6.tanf_cancelled_identify_hh_indcase_spells AS \n",
    "SELECT *, 1 as tag_cancel_tanf\n",
    "FROM(\n",
    "SELECT cancel.ch_dpa_caseid, cancel.transaction_date,\n",
    "spell.recptno, spell.benefit_type, spell.end_date , \n",
    "(cancel.transaction_date-spell.end_date) as days_between_end_cancel,\n",
    "FIRST_VALUE(spell.end_date) \n",
    "OVER (PARTITION BY cancel.transaction_date \n",
    "ORDER BY ABS(cancel.transaction_date-spell.end_date))\n",
    "FROM idhs.tanf_cancelled_earnings_redacted cancel\n",
    "LEFT JOIN idhs.hh_indcase_spells spell\n",
    "ON cancel.ch_dpa_caseid=spell.ch_dpa_caseid\n",
    "WHERE (cancel.ch_dpa_caseid IS NOT NULL) AND \n",
    "(spell.benefit_type LIKE '%tanf%')\n",
    "order by cancel.ch_dpa_caseid,spell.end_date\n",
    ") temp\n",
    "WHERE ABS(days_between_end_cancel)<=60\n",
    "ORDER BY ABS(days_between_end_cancel)"
   ]
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 2",
   "language": "python",
   "name": "python2"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 2
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython2",
   "version": "2.7.12"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 2
}