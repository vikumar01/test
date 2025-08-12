df_a['abc'] = df_a['id'].map(df_b.set_index('id')['abc']).fillna(df_a['abc'])
