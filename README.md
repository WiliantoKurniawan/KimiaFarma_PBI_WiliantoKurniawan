# Kimia-Farma---Final-Task---PBI---Wilianto-Kurniawan

#membuat CTE untuk melakukan kategorisasi persentase gross laba berdasarkan kelompok harga 
WITH gross_laba AS(
  SELECT 
    *,
    CASE 
      WHEN prd.price <= 50000 THEN 0.1
      WHEN prd.price >50000 AND prd.price <=100000 THEN 0.15
      WHEN prd.price >100000 AND prd.price <=300000 THEN 0.2
      WHEN prd.price >300000 AND prd.price <=500000 THEN 0.25
      ELSE 0.3
    END AS persentase_gross_laba
  FROM `rakamin-kf-analytics-416103.kimia_farma.kf_product` prd
)

SELECT 
  trx.transaction_id, trx.date, trx.customer_name, trx.branch_id,
  cbg.branch_name, cbg.kota, cbg.provinsi,cbg.rating AS rating_cabang,
  prd.product_id, prd.product_name, prd.price AS actual_price,
  trx.discount_percentage,
  gl.persentase_gross_laba,
  (1-trx.discount_percentage)*prd.price AS net_sales,
  (gl.persentase_gross_laba-trx.discount_percentage)*prd.price AS net_profit,
  trx.rating AS rating_transaksi
FROM `rakamin-kf-analytics-416103.kimia_farma.kf_final_transaction`AS trx 
JOIN `rakamin-kf-analytics-416103.kimia_farma.kf_kantor_cabang`AS cbg ON trx.branch_id = cbg.branch_id
JOIN `rakamin-kf-analytics-416103.kimia_farma.kf_product`AS prd ON trx.product_id = prd.product_id
JOIN gross_laba AS gl ON trx.product_id = gl.product_id
